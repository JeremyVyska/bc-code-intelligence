# AL Facade Pattern for External API Integration - Examples

## Basic Facade Structure

```al
// Facade Pattern: External API Integration Manager
// Provides simplified interface for complex external API operations
// Centralizes authentication, error handling, and business logic
codeunit 50300 "External API Manager"
{
    // Public business methods - simplified interface for consumers
    procedure GetCustomerData(CustomerNo: Code[20]; var CustomerData: Record "Customer"): Boolean
    var
        JsonResponse: JsonObject;
    begin
        // Business-focused method signature - hides API complexity
        if not CallCustomerAPI(CustomerNo, JsonResponse) then
            exit(false);

        exit(ParseCustomerResponse(JsonResponse, CustomerData));
    end;

    procedure SyncOrderStatus(SalesOrderNo: Code[20]): Boolean
    var
        JsonResponse: JsonObject;
        OrderStatus: Text;
    begin
        // Single method for complex order synchronization process
        if not CallOrderStatusAPI(SalesOrderNo, JsonResponse) then
            exit(false);

        if not ExtractOrderStatus(JsonResponse, OrderStatus) then
            exit(false);

        exit(UpdateLocalOrderStatus(SalesOrderNo, OrderStatus));
    end;

    procedure ValidateProductCatalog(ItemNo: Code[20]; var IsValid: Boolean; var ErrorMessage: Text): Boolean
    var
        JsonResponse: JsonObject;
    begin
        // Centralized validation with business-friendly error messages
        if not CallProductValidationAPI(ItemNo, JsonResponse) then begin
            ErrorMessage := 'Unable to validate product with external system';
            IsValid := false;
            exit(false);
        end;

        exit(ProcessValidationResponse(JsonResponse, IsValid, ErrorMessage));
    end;

    // Private implementation methods - hidden complexity
    local procedure CallCustomerAPI(CustomerNo: Code[20]; var JsonResponse: JsonObject): Boolean
    var
        HttpClient: HttpClient;
        RequestMessage: HttpRequestMessage;
        ResponseMessage: HttpResponseMessage;
        ApiUrl: Text;
    begin
        // Handle authentication automatically
        if not EnsureAuthenticated() then
            exit(false);

        // Construct API endpoint with proper formatting
        ApiUrl := StrSubstNo('%1/customers/%2', GetBaseApiUrl(), CustomerNo);

        // Configure authenticated request
        if not PrepareAuthenticatedRequest(RequestMessage, 'GET', ApiUrl) then
            exit(false);

        // Execute with retry logic and error handling
        if not ExecuteApiCall(HttpClient, RequestMessage, ResponseMessage) then
            exit(false);

        exit(ParseJsonResponse(ResponseMessage, JsonResponse));
    end;
}
```

## API Call Abstraction

```al
// Private methods handling technical API details
codeunit 50301 "API Technical Layer"
{
    var
        AuthToken: Text;
        BaseApiUrl: Text;
        LastAuthTime: DateTime;
        TokenExpiryMinutes: Integer;

    // Authentication management - hidden from business logic
    local procedure EnsureAuthenticated(): Boolean
    var
        CurrentTime: DateTime;
        MinutesSinceAuth: Integer;
    begin
        CurrentTime := CurrentDateTime;
        MinutesSinceAuth := (CurrentTime - LastAuthTime) / 60000;

        // Token refresh logic
        if (AuthToken = '') or (MinutesSinceAuth >= TokenExpiryMinutes) then
            exit(RefreshAuthToken());

        exit(AuthToken <> '');
    end;

    local procedure RefreshAuthToken(): Boolean
    var
        HttpClient: HttpClient;
        RequestMessage: HttpRequestMessage;
        ResponseMessage: HttpResponseMessage;
        AuthEndpoint: Text;
        RequestBody: Text;
        Content: HttpContent;
        JsonResponse: JsonObject;
        TokenValue: JsonToken;
        Headers: HttpHeaders;
    begin
        AuthEndpoint := GetBaseApiUrl() + '/auth/token';
        RequestBody := StrSubstNo('{"client_id": "%1", "client_secret": "%2", "grant_type": "client_credentials"}',
                                 GetClientId(), GetClientSecret());

        Content.WriteFrom(RequestBody);
        Content.GetHeaders(Headers);
        Headers.Clear();
        Headers.Add('Content-Type', 'application/json');

        RequestMessage.Method := 'POST';
        RequestMessage.SetRequestUri(AuthEndpoint);
        RequestMessage.Content := Content;

        if not HttpClient.Send(RequestMessage, ResponseMessage) then begin
            LogError('Authentication request failed');
            exit(false);
        end;

        if not ResponseMessage.IsSuccessStatusCode() then begin
            LogError(StrSubstNo('Authentication failed with status: %1', ResponseMessage.HttpStatusCode()));
            exit(false);
        end;

        if not ParseJsonResponse(ResponseMessage, JsonResponse) then
            exit(false);

        if JsonResponse.Get('access_token', TokenValue) then begin
            AuthToken := TokenValue.AsValue().AsText();
            LastAuthTime := CurrentDateTime;
            TokenExpiryMinutes := 55; // Refresh before 60-minute expiry
            exit(true);
        end;

        LogError('Access token not found in authentication response');
        exit(false);
    end;

    // Generic API call execution with retry logic
    local procedure ExecuteApiCall(var HttpClient: HttpClient; RequestMessage: HttpRequestMessage; var ResponseMessage: HttpResponseMessage): Boolean
    var
        RetryCount: Integer;
        MaxRetries: Integer;
        Success: Boolean;
        StatusCode: Integer;
    begin
        MaxRetries := 3;
        RetryCount := 0;

        repeat
            Success := HttpClient.Send(RequestMessage, ResponseMessage);

            if Success then begin
                StatusCode := ResponseMessage.HttpStatusCode();

                // Success status codes
                if (StatusCode >= 200) and (StatusCode < 300) then
                    exit(true);

                // Retryable status codes (temporary failures)
                if (StatusCode = 429) or (StatusCode >= 500) then begin
                    RetryCount += 1;
                    if RetryCount < MaxRetries then begin
                        Sleep(RetryCount * 2000); // Exponential backoff: 2s, 4s, 6s
                        continue;
                    end;
                end;
            end;

            RetryCount += 1;
            if RetryCount < MaxRetries then
                Sleep(RetryCount * 2000);

        until RetryCount >= MaxRetries;

        LogError(StrSubstNo('API call failed after %1 retries', MaxRetries));
        exit(false);
    end;
}
```

## Error Handling and Authentication

```al
// Centralized error handling with business-friendly messages
codeunit 50302 "API Error Handler"
{
    // Business-friendly error handling
    procedure HandleApiError(var ErrorMessage: Text; HttpStatusCode: Integer; ApiResponse: Text): Boolean
    var
        JsonResponse: JsonObject;
        ErrorToken: JsonToken;
        ErrorCode: Text;
        DetailedMessage: Text;
    begin
        // Parse error response if JSON
        if JsonResponse.ReadFrom(ApiResponse) then begin
            if JsonResponse.Get('error_code', ErrorToken) then
                ErrorCode := ErrorToken.AsValue().AsText();

            if JsonResponse.Get('message', ErrorToken) then
                DetailedMessage := ErrorToken.AsValue().AsText();
        end;

        // Convert technical errors to business messages
        case HttpStatusCode of
            400:
                ErrorMessage := GetBusinessErrorMessage('INVALID_REQUEST', ErrorCode, DetailedMessage);
            401:
                ErrorMessage := 'Authentication failed. Please check system configuration.';
            403:
                ErrorMessage := 'Access denied. Insufficient permissions for this operation.';
            404:
                ErrorMessage := 'The requested resource was not found in the external system.';
            409:
                ErrorMessage := 'Data conflict detected. The record may have been modified by another user.';
            422:
                ErrorMessage := GetBusinessErrorMessage('VALIDATION_FAILED', ErrorCode, DetailedMessage);
            429:
                ErrorMessage := 'External system is busy. Please try again in a few minutes.';
            500, 502, 503:
                ErrorMessage := 'External system is temporarily unavailable. Please try again later.';
            else
                ErrorMessage := StrSubstNo('Unexpected error occurred (Code: %1). Please contact your administrator.', HttpStatusCode);
        end;

        // Log technical details for troubleshooting (not shown to user)
        LogTechnicalError(HttpStatusCode, ApiResponse);

        exit(false); // Always return false for error conditions
    end;
}
```

## Anti-Pattern: Direct API Calls

```al
// ANTI-PATTERN: What NOT to do - Scattered API calls throughout codebase
// Problems: Code duplication, inconsistent error handling, tight coupling

// BAD EXAMPLE: Direct API calls in business logic
codeunit 50400 "Sales Order Processing" // DON'T DO THIS
{
    procedure ProcessSalesOrder(var SalesHeader: Record "Sales Header")
    var
        HttpClient: HttpClient;
        RequestMessage: HttpRequestMessage;
        ResponseMessage: HttpResponseMessage;
        CustomerNo: Code[20];
        AuthToken: Text;
        ApiUrl: Text;
    begin
        CustomerNo := SalesHeader."Sell-to Customer No.";

        // PROBLEM: Authentication logic scattered everywhere
        AuthToken := GetAuthTokenSomehow(); // Duplicated in multiple places

        // PROBLEM: URL construction repeated in many methods
        ApiUrl := 'https://api.external.com/customers/' + CustomerNo;

        RequestMessage.Method := 'GET';
        RequestMessage.SetRequestUri(ApiUrl);

        // PROBLEM: No standardized error handling
        if not HttpClient.Send(RequestMessage, ResponseMessage) then
            Error('Something went wrong'); // Unhelpful error message

        // Business logic mixed with technical concerns
        ProcessCustomerData(ResponseMessage);
    end;
}

/*
PROBLEMS WITH ANTI-PATTERN APPROACH:

1. Code Duplication:
   - Authentication logic repeated in every method
   - URL construction patterns scattered throughout
   - Response parsing duplicated everywhere

2. Maintenance Nightmare:
   - Changes to API authentication require updates in multiple places
   - URL changes affect many different files
   - Error handling improvements need to be replicated everywhere

3. Poor Separation of Concerns:
   - Business logic mixed with technical HTTP details
   - UI components directly calling APIs
   - Reports tightly coupled to external systems

4. Testing Difficulties:
   - Cannot easily mock external API calls
   - Business logic testing requires network setup
   - Integration tests become complex and brittle

CORRECT APPROACH: Use the Facade Pattern shown above
- Centralized API management
- Consistent error handling
- Separated concerns
- Testable components
*/
```