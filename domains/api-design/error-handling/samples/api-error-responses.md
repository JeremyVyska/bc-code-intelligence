# Business Central API Error Response Patterns - AL Code Sample

## Basic API Error Handling

```al
page 50120 "Customer API Error Handling"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'customers';
    APIVersion = 'v1.0';
    EntityName = 'customer';
    EntitySetName = 'customers';
    SourceTable = Customer;
    DelayedInsert = true;
    
    layout
    {
        area(Content)
        {
            repeater(Records)
            {
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                }
                
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                    trigger OnValidate()
                    begin
                        // Validation error - returns 400 Bad Request
                        if Rec."No." = '' then
                            Error('Customer number cannot be empty');
                            
                        if StrLen(Rec."No.") > 20 then
                            Error('Customer number cannot exceed 20 characters');
                            
                        // Check for duplicates - returns 400 Bad Request
                        if CustomerExists(Rec."No.") then
                            Error('Customer with number %1 already exists', Rec."No.");
                    end;
                }
                
                field(name; Rec.Name)
                {
                    Caption = 'Name';
                    trigger OnValidate()
                    begin
                        // Required field validation
                        if Rec.Name = '' then
                            Error('Customer name is required');
                    end;
                }
                
                field(creditLimit; Rec."Credit Limit (LCY)")
                {
                    Caption = 'Credit Limit';
                    trigger OnValidate()
                    begin
                        // Business logic validation
                        if Rec."Credit Limit (LCY)" < 0 then
                            Error('Credit limit cannot be negative');
                            
                        // Custom business rule
                        if (Rec."Credit Limit (LCY)" > 100000) and (Rec."Customer Posting Group" <> 'WHOLESALE') then
                            Error('Credit limit above 100,000 requires wholesale customer posting group');
                    end;
                }
                
                field(blocked; Rec.Blocked)
                {
                    Caption = 'Blocked';
                }
            }
        }
    }
    
    // Custom validation procedures
    local procedure CustomerExists(CustomerNo: Code[20]): Boolean
    var
        Customer: Record Customer;
    begin
        Customer.SetRange("No.", CustomerNo);
        exit(not Customer.IsEmpty);
    end;
}
```

## Advanced Error Handling with Custom Messages

```al
codeunit 50120 "API Error Management"
{
    // Helper procedures for consistent API error handling
    
    procedure ValidateCustomerForAPI(var Customer: Record Customer)
    begin
        // Comprehensive customer validation
        ValidateRequiredFields(Customer);
        ValidateBusinessRules(Customer);
        ValidateDataIntegrity(Customer);
    end;
    
    local procedure ValidateRequiredFields(var Customer: Record Customer)
    begin
        if Customer."No." = '' then
            ThrowValidationError('MISSING_CUSTOMER_NUMBER', 'Customer number is required');
            
        if Customer.Name = '' then
            ThrowValidationError('MISSING_CUSTOMER_NAME', 'Customer name is required');
            
        if Customer."Customer Posting Group" = '' then
            ThrowValidationError('MISSING_POSTING_GROUP', 'Customer posting group is required');
    end;
    
    local procedure ValidateBusinessRules(var Customer: Record Customer)
    var
        CustomerPostingGroup: Record "Customer Posting Group";
        PaymentTerms: Record "Payment Terms";
    begin
        // Validate posting group exists
        if not CustomerPostingGroup.Get(Customer."Customer Posting Group") then
            ThrowValidationError('INVALID_POSTING_GROUP', 
                StrSubstNo('Customer posting group %1 does not exist', Customer."Customer Posting Group"));
                
        // Validate payment terms if specified
        if Customer."Payment Terms Code" <> '' then
            if not PaymentTerms.Get(Customer."Payment Terms Code") then
                ThrowValidationError('INVALID_PAYMENT_TERMS',
                    StrSubstNo('Payment terms %1 do not exist', Customer."Payment Terms Code"));
                    
        // Credit limit business rules
        if Customer."Credit Limit (LCY)" > 1000000 then
            ThrowValidationError('CREDIT_LIMIT_EXCEEDED',
                'Credit limit cannot exceed 1,000,000 without special approval');
    end;
    
    local procedure ValidateDataIntegrity(var Customer: Record Customer)
    var
        ExistingCustomer: Record Customer;
    begin
        // Check for duplicate customer names in same posting group
        ExistingCustomer.SetRange("Customer Posting Group", Customer."Customer Posting Group");
        ExistingCustomer.SetRange(Name, Customer.Name);
        ExistingCustomer.SetFilter("No.", '<>%1', Customer."No.");
        
        if not ExistingCustomer.IsEmpty then
            ThrowValidationError('DUPLICATE_CUSTOMER_NAME',
                StrSubstNo('Customer with name %1 already exists in posting group %2', 
                    Customer.Name, Customer."Customer Posting Group"));
    end;
    
    local procedure ThrowValidationError(ErrorCode: Text; ErrorMessage: Text)
    begin
        // This creates a structured error response
        // BC automatically wraps this in proper OData error format
        Error('API_VALIDATION_ERROR:%1:%2', ErrorCode, ErrorMessage);
    end;
}

// Usage in API page
page 50121 "Customer API Enhanced"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'customers';
    APIVersion = 'v2.0';
    EntityName = 'customer';
    EntitySetName = 'customers';
    SourceTable = Customer;
    DelayedInsert = true;
    
    layout
    {
        area(Content)
        {
            repeater(Records)
            {
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                }
                
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                }
                
                field(name; Rec.Name)
                {
                    Caption = 'Name';
                }
                
                field(customerPostingGroup; Rec."Customer Posting Group")
                {
                    Caption = 'Customer Posting Group';
                }
                
                field(creditLimit; Rec."Credit Limit (LCY)")
                {
                    Caption = 'Credit Limit';
                }
            }
        }
    }
    
    var
        APIErrorMgt: Codeunit "API Error Management";
    
    trigger OnInsertRecord(BelowxRec: Boolean): Boolean
    begin
        // Validate before insert - returns 400 if validation fails
        APIErrorMgt.ValidateCustomerForAPI(Rec);
        exit(true);
    end;
    
    trigger OnModifyRecord(): Boolean
    begin
        // Validate before modify - returns 400 if validation fails
        APIErrorMgt.ValidateCustomerForAPI(Rec);
        exit(true);
    end;
    
    trigger OnDeleteRecord(): Boolean
    begin
        // Check if customer can be deleted - returns 409 if conflict
        CheckCustomerCanBeDeleted();
        exit(true);
    end;
    
    local procedure CheckCustomerCanBeDeleted()
    var
        CustLedgerEntry: Record "Cust. Ledger Entry";
        SalesHeader: Record "Sales Header";
    begin
        // Check for open transactions
        CustLedgerEntry.SetRange("Customer No.", Rec."No.");
        CustLedgerEntry.SetRange(Open, true);
        if not CustLedgerEntry.IsEmpty then
            Error('CUSTOMER_HAS_OPEN_ENTRIES:Cannot delete customer with open ledger entries');
            
        // Check for pending orders
        SalesHeader.SetRange("Sell-to Customer No.", Rec."No.");
        SalesHeader.SetFilter("Document Type", '%1|%2', SalesHeader."Document Type"::Order, SalesHeader."Document Type"::Invoice);
        if not SalesHeader.IsEmpty then
            Error('CUSTOMER_HAS_PENDING_ORDERS:Cannot delete customer with pending sales orders');
    end;
}
```

## HTTP Status Code Mapping and Error Categories

```al
codeunit 50121 "API Error Response Handler"
{
    // Demonstrates how different AL errors map to HTTP status codes
    
    procedure DemonstrateErrorTypes(var Customer: Record Customer)
    begin
        // 400 Bad Request - Validation errors
        ValidateRequiredData(Customer);
        
        // 404 Not Found - Record not found
        ValidateRecordExists(Customer);
        
        // 409 Conflict - Business rule conflicts
        ValidateBusinessConflicts(Customer);
        
        // 422 Unprocessable Entity - Semantic errors
        ValidateSemanticRules(Customer);
    end;
    
    local procedure ValidateRequiredData(var Customer: Record Customer)
    begin
        // These errors result in 400 Bad Request
        if Customer."No." = '' then
            Error('Customer number is required');  // 400
            
        if Customer.Name = '' then
            Error('Customer name cannot be empty');  // 400
            
        if StrLen(Customer."No.") > 20 then
            Error('Customer number exceeds maximum length of 20 characters');  // 400
    end;
    
    local procedure ValidateRecordExists(var Customer: Record Customer)
    var
        CustomerPostingGroup: Record "Customer Posting Group";
        PaymentTerms: Record "Payment Terms";
    begin
        // These errors result in 400 Bad Request (not 404 - BC limitation)
        if Customer."Customer Posting Group" <> '' then
            if not CustomerPostingGroup.Get(Customer."Customer Posting Group") then
                Error('Customer posting group %1 does not exist', Customer."Customer Posting Group");
                
        if Customer."Payment Terms Code" <> '' then
            if not PaymentTerms.Get(Customer."Payment Terms Code") then
                Error('Payment terms %1 do not exist', Customer."Payment Terms Code");
    end;
    
    local procedure ValidateBusinessConflicts(var Customer: Record Customer)
    var
        ExistingCustomer: Record Customer;
    begin
        // These errors typically result in 400 Bad Request
        // BC doesn't directly support 409 Conflict, but these are conceptually conflicts
        
        ExistingCustomer.SetRange("No.", Customer."No.");
        ExistingCustomer.SetFilter(SystemId, '<>%1', Customer.SystemId);
        if not ExistingCustomer.IsEmpty then
            Error('Customer number %1 already exists', Customer."No.");  // Conceptually 409
    end;
    
    local procedure ValidateSemanticRules(var Customer: Record Customer)
    begin
        // Complex business rule validation - 400 Bad Request
        if (Customer."Credit Limit (LCY)" > 50000) and (Customer."Payment Terms Code" = '') then
            Error('High credit limit customers must have payment terms specified');
            
        if Customer.Blocked <> Customer.Blocked::" " then
            if Customer."Credit Limit (LCY)" > 0 then
                Error('Blocked customers cannot have credit limit greater than zero');
    end;
    
    // Error formatting for consistent API responses
    procedure FormatAPIError(ErrorCode: Text; ErrorMessage: Text; ErrorCategory: Text): Text
    var
        ErrorJsonObject: JsonObject;
        ErrorJson: Text;
    begin
        // Create structured error response
        ErrorJsonObject.Add('code', ErrorCode);
        ErrorJsonObject.Add('message', ErrorMessage);
        ErrorJsonObject.Add('category', ErrorCategory);
        ErrorJsonObject.Add('timestamp', CurrentDateTime);
        ErrorJsonObject.WriteTo(ErrorJson);
        
        exit(ErrorJson);
    end;
}
```

## Error Response Examples and Testing

```al
codeunit 50122 "API Error Testing"
{
    // Test various error scenarios to understand response formats
    
    [Test]
    procedure TestValidationErrors()
    var
        Customer: Record Customer;
        APIErrorHandler: Codeunit "API Error Response Handler";
    begin
        // Test required field validation
        Customer.Init();
        Customer."No." := '';  // Missing required field
        
        asserterror APIErrorHandler.DemonstrateErrorTypes(Customer);
        
        // Expected response: 400 Bad Request
        // {
        //   "error": {
        //     "code": "BadRequest",
        //     "message": "Customer number is required"
        //   }
        // }
        
        Assert.ExpectedError('Customer number is required');
    end;
    
    [Test]
    procedure TestBusinessRuleViolation()
    var
        Customer: Record Customer;
        APIErrorHandler: Codeunit "API Error Response Handler";
    begin
        // Test business rule validation
        Customer.Init();
        Customer."No." := 'TEST001';
        Customer.Name := 'Test Customer';
        Customer."Credit Limit (LCY)" := 75000;
        Customer."Payment Terms Code" := '';  // Violates business rule
        
        asserterror APIErrorHandler.DemonstrateErrorTypes(Customer);
        
        // Expected response: 400 Bad Request
        // {
        //   "error": {
        //     "code": "BadRequest", 
        //     "message": "High credit limit customers must have payment terms specified"
        //   }
        // }
        
        Assert.ExpectedError('High credit limit customers must have payment terms specified');
    end;
    
    [Test]
    procedure TestRecordNotFound()
    var
        Customer: Record Customer;
        APIErrorHandler: Codeunit "API Error Response Handler";
    begin
        // Test reference to non-existent record
        Customer.Init();
        Customer."No." := 'TEST002';
        Customer.Name := 'Test Customer';
        Customer."Customer Posting Group" := 'NONEXISTENT';
        
        asserterror APIErrorHandler.DemonstrateErrorTypes(Customer);
        
        // Expected response: 400 Bad Request (BC limitation - should be 404)
        // {
        //   "error": {
        //     "code": "BadRequest",
        //     "message": "Customer posting group NONEXISTENT does not exist"
        //   }
        // }
        
        Assert.ExpectedError('Customer posting group NONEXISTENT does not exist');
    end;
}

// Error response interceptor for custom formatting
codeunit 50123 "API Error Formatter"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"API Error Publisher", 'OnAfterPublishError', '', false, false)]
    local procedure OnAfterPublishAPIError(ErrorContext: Text; var ErrorPayload: Text)
    var
        ErrorDetails: JsonObject;
        CustomError: JsonObject;
    begin
        // Intercept and format API errors for consistent response structure
        if ErrorDetails.ReadFrom(ErrorPayload) then begin
            CustomError.Add('timestamp', CurrentDateTime);
            CustomError.Add('path', ErrorContext);
            CustomError.Add('error', ErrorDetails);
            CustomError.Add('apiVersion', 'v2.0');
            
            CustomError.WriteTo(ErrorPayload);
        end;
    end;
}
```

## Implementation Notes

**BC API Error Response Behavior:**
- Most AL errors return 400 Bad Request status code
- BC automatically wraps AL Error() calls in OData error format
- Custom error codes can be embedded in error messages
- Error details include message, code, and sometimes inner error information

**Error Categories and HTTP Status Mapping:**
- **400 Bad Request**: Validation errors, missing required fields, format errors
- **401 Unauthorized**: Authentication failures (handled by BC framework)
- **403 Forbidden**: Permission denied (handled by BC framework)  
- **404 Not Found**: Invalid API endpoint (handled by BC framework)
- **409 Conflict**: Not directly supported - use 400 with descriptive message
- **422 Unprocessable Entity**: Not directly supported - use 400
- **500 Internal Server Error**: Unexpected AL runtime errors

**Best Practices:**
- Use descriptive error messages that help API consumers
- Include error codes for programmatic error handling
- Validate data comprehensively before database operations
- Handle business rule violations gracefully
- Test error scenarios thoroughly
- Consider creating custom error response formats for better API experience

**Testing Error Responses:**
- Use BC Test Toolkit to validate error handling
- Test all validation scenarios systematically
- Verify error message clarity and usefulness
- Ensure consistent error format across API endpoints
- Monitor error logs for unexpected error patterns