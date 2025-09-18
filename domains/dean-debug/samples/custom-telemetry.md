# Custom Telemetry Implementation - AL Code Examples

## Basic Session.LogMessage() Implementation

```al
// Basic telemetry logging with Session.LogMessage()
codeunit 50300 "Basic Telemetry Examples"
{
    procedure LogBasicEvent()
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        // Simple event logging without custom dimensions
        Session.LogMessage('0001', 'User accessed customer list', Verbosity::Normal,
                          DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, '');

        // Event logging with basic custom dimensions
        CustomDimensions.Add('FeatureName', 'CustomerManagement');
        CustomDimensions.Add('ActionType', 'ViewList');

        Session.LogMessage('0002', 'Customer list accessed with filters', Verbosity::Normal,
                          DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                          CustomDimensions);
    end;
}
```

## Privacy-Compliant Telemetry Patterns

```al
// Privacy-compliant telemetry implementation
codeunit 50301 "Privacy Safe Telemetry"
{
    procedure LogUserActionSafely(CustomerNo: Code[20]; ActionPerformed: Text)
    var
        CustomDimensions: Dictionary of [Text, Text];
        HashedCustomerNo: Text;
    begin
        // CORRECT: Hash or anonymize sensitive data
        HashedCustomerNo := CreateGuid(); // Simplified example - use proper hashing in production

        CustomDimensions.Add('CustomerHash', HashedCustomerNo);
        CustomDimensions.Add('ActionType', ActionPerformed);
        CustomDimensions.Add('CompanySize', GetCompanySizeCategory());
        CustomDimensions.Add('UserRole', GetUserRoleCategory());

        Session.LogMessage('0010', 'Customer action performed', Verbosity::Normal,
                          DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                          CustomDimensions);
    end;

    procedure LogSalesDocumentEvent(DocumentType: Enum "Sales Document Type"; LineCount: Integer)
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        // CORRECT: Log aggregated, non-sensitive data
        CustomDimensions.Add('DocumentType', Format(DocumentType));
        CustomDimensions.Add('LineCountRange', GetLineCountRange(LineCount));
        CustomDimensions.Add('ProcessingTime', Format(CurrentDateTime - StartTime));

        Session.LogMessage('0011', 'Sales document processed', Verbosity::Normal,
                          DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                          CustomDimensions);
    end;

    local procedure GetCompanySizeCategory(): Text
    var
        CustomerCount: Integer;
        Customer: Record Customer;
    begin
        // Categorize without exposing exact counts
        CustomerCount := Customer.Count();

        case CustomerCount of
            0..50: exit('Small');
            51..500: exit('Medium');
            else exit('Large');
        end;
    end;

    local procedure GetLineCountRange(LineCount: Integer): Text
    begin
        // Return ranges instead of exact counts for privacy
        case LineCount of
            0..5: exit('1-5');
            6..20: exit('6-20');
            21..50: exit('21-50');
            else exit('50+');
        end;
    end;
}
```

## Error Handling and Performance Monitoring

```al
// Error handling and performance telemetry
codeunit 50302 "Error and Performance Telemetry"
{
    procedure ProcessWithTelemetry()
    var
        CustomDimensions: Dictionary of [Text, Text];
        StartTime: DateTime;
        ProcessingDuration: Duration;
        IsSuccess: Boolean;
    begin
        StartTime := CurrentDateTime;

        // Log process start
        CustomDimensions.Add('ProcessName', 'DataImport');
        CustomDimensions.Add('StartTime', Format(StartTime));

        Session.LogMessage('0020', 'Data import process started', Verbosity::Normal,
                          DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                          CustomDimensions);

        // Perform the actual work with error handling
        IsSuccess := TryExecuteProcess();
        ProcessingDuration := CurrentDateTime - StartTime;

        // Log completion with performance metrics
        Clear(CustomDimensions);
        CustomDimensions.Add('ProcessName', 'DataImport');
        CustomDimensions.Add('Duration', Format(ProcessingDuration));
        CustomDimensions.Add('Success', Format(IsSuccess));
        CustomDimensions.Add('PerformanceCategory', GetPerformanceCategory(ProcessingDuration));

        if IsSuccess then begin
            Session.LogMessage('0021', 'Data import completed successfully', Verbosity::Normal,
                              DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                              CustomDimensions);
        end else begin
            // Add error context without exposing sensitive data
            CustomDimensions.Add('ErrorCategory', GetLastErrorCategory());
            Session.LogMessage('0022', 'Data import failed', Verbosity::Error,
                              DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                              CustomDimensions);
        end;
    end;

    [TryFunction]
    local procedure TryExecuteProcess()
    begin
        // Simulate work that might fail
        if Random(10) > 7 then
            Error('Simulated processing error');
    end;

    local procedure GetPerformanceCategory(Duration: Duration): Text
    begin
        // Categorize performance for analysis
        if Duration < 1000 then exit('Fast');
        if Duration < 5000 then exit('Normal');
        if Duration < 15000 then exit('Slow');
        exit('VerySlow');
    end;

    local procedure GetLastErrorCategory(): Text
    var
        ErrorText: Text;
    begin
        ErrorText := GetLastErrorText();

        // Categorize errors without exposing sensitive details
        if StrPos(ErrorText, 'permission') > 0 then exit('Permission');
        if StrPos(ErrorText, 'network') > 0 then exit('Network');
        if StrPos(ErrorText, 'database') > 0 then exit('Database');
        if StrPos(ErrorText, 'timeout') > 0 then exit('Timeout');
        exit('General');
    end;
}
```

## API Usage Telemetry

```al
// API performance and usage tracking
codeunit 50303 "API Usage Telemetry"
{
    procedure LogAPICall(APIEndpoint: Text; ResponseTime: Duration; Success: Boolean)
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        // Track API performance and usage patterns
        CustomDimensions.Add('APIEndpoint', APIEndpoint);
        CustomDimensions.Add('ResponseTime', Format(ResponseTime));
        CustomDimensions.Add('Success', Format(Success));
        CustomDimensions.Add('PerformanceTier', GetAPIPerformanceTier(ResponseTime));
        CustomDimensions.Add('TimeOfDay', GetTimeCategory());

        Session.LogMessage('0030', 'API call completed', Verbosity::Normal,
                          DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                          CustomDimensions);
    end;

    procedure LogFeatureUsage(FeatureName: Text; FeatureVersion: Text; UsageContext: Text)
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        // Track feature adoption and usage patterns
        CustomDimensions.Add('FeatureName', FeatureName);
        CustomDimensions.Add('FeatureVersion', FeatureVersion);
        CustomDimensions.Add('UsageContext', UsageContext);
        CustomDimensions.Add('UserCategory', GetAnonymousUserCategory());
        CustomDimensions.Add('SessionId', GetSessionHash());

        Session.LogMessage('0031', 'Feature used', Verbosity::Normal,
                          DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                          CustomDimensions);
    end;

    local procedure GetAPIPerformanceTier(ResponseTime: Duration): Text
    begin
        if ResponseTime < 500 then exit('Excellent');
        if ResponseTime < 2000 then exit('Good');
        if ResponseTime < 5000 then exit('Acceptable');
        exit('Poor');
    end;

    local procedure GetTimeCategory(): Text
    var
        CurrentTime: Time;
    begin
        CurrentTime := Time;
        if CurrentTime < 090000T then exit('EarlyMorning');
        if CurrentTime < 120000T then exit('Morning');
        if CurrentTime < 140000T then exit('Midday');
        if CurrentTime < 170000T then exit('Afternoon');
        exit('Evening');
    end;

    local procedure GetAnonymousUserCategory(): Text
    var
        UserSetup: Record "User Setup";
    begin
        // Categorize users without identifying them
        if UserSetup.Get(UserId) then begin
            if UserSetup."Sales Resp. Ctr. Filter" <> '' then exit('SalesUser');
            if UserSetup."Purchase Resp. Ctr. Filter" <> '' then exit('PurchaseUser');
        end;
        exit('GeneralUser');
    end;

    local procedure GetSessionHash(): Text
    begin
        // Create session identifier without exposing user identity
        exit(CopyStr(CreateGuid(), 1, 8));
    end;
}
```

## Anti-Pattern Examples (What NOT to Do)

```al
// ANTI-PATTERNS: Examples of incorrect telemetry implementation
codeunit 50304 "Telemetry Anti-Patterns"
{
    // ❌ WRONG: Logging sensitive customer data
    procedure BadExample_SensitiveData(CustomerNo: Code[20]; CustomerName: Text)
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        // ❌ NEVER log personal or sensitive business data
        CustomDimensions.Add('CustomerNo', CustomerNo);        // Exposes customer identity
        CustomDimensions.Add('CustomerName', CustomerName);    // Exposes personal data
        CustomDimensions.Add('CreditLimit', '50000');          // Exposes business sensitive data

        // This violates privacy regulations and security best practices
        Session.LogMessage('9001', 'Customer accessed - BAD EXAMPLE', Verbosity::Normal,
                          DataClassification::CustomerContent,  // Wrong classification
                          TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // ❌ WRONG: Excessive logging that impacts performance
    procedure BadExample_ExcessiveLogging()
    var
        Customer: Record Customer;
        CustomDimensions: Dictionary of [Text, Text];
    begin
        // ❌ NEVER log inside tight loops without consideration
        Customer.FindSet();
        repeat
            CustomDimensions.Add('CustomerProcessed', Customer."No.");  // Also privacy violation
            // This will create thousands of telemetry events and impact performance
            Session.LogMessage('9002', 'Processing customer', Verbosity::Verbose,
                              DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                              CustomDimensions);
        until Customer.Next() = 0;
    end;

    // ❌ WRONG: Poor error logging without context
    procedure BadExample_PoorErrorLogging()
    begin
        // ❌ Logging error without useful context for debugging
        Session.LogMessage('9003', 'An error occurred', Verbosity::Error,
                          DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, '');

        // Missing: Error category, operation context, environmental factors
    end;

    // ❌ WRONG: Using wrong telemetry scope and classification
    procedure BadExample_WrongScope()
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions.Add('InternalProcessId', CreateGuid());

        // ❌ Wrong scope - should be ExtensionPublisher for app-specific events
        // ❌ Wrong classification - internal process data shouldn't be CustomerContent
        Session.LogMessage('9004', 'Internal process', Verbosity::Normal,
                          DataClassification::CustomerContent,        // Wrong classification
                          TelemetryScope::All,                        // Wrong scope
                          CustomDimensions);
    end;
}
```

## Performance-Conscious Telemetry

```al
// Performance-aware telemetry implementation
codeunit 50305 "Performance Aware Telemetry"
{
    var
        SamplingRate: Integer;
        EventCounter: Integer;

    procedure InitializeTelemetry()
    begin
        SamplingRate := 5; // 5% sampling for high-frequency events
        EventCounter := 0;
    end;

    procedure LogHighFrequencyEvent(EventType: Text)
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        EventCounter += 1;

        // Use sampling to reduce telemetry volume
        if (EventCounter mod (100 div SamplingRate)) = 0 then begin
            CustomDimensions.Add('EventType', EventType);
            CustomDimensions.Add('SamplingRate', Format(SamplingRate) + '%');
            CustomDimensions.Add('EventsSinceLastLog', Format(100 div SamplingRate));

            Session.LogMessage('PERF001', 'Sampled high-frequency event', Verbosity::Normal,
                              DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                              CustomDimensions);
        end;
    end;

    procedure LogBatchOperation(OperationType: Text; RecordCount: Integer; Duration: Duration)
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        // Aggregate telemetry for batch operations
        CustomDimensions.Add('OperationType', OperationType);
        CustomDimensions.Add('RecordCountRange', GetRecordCountRange(RecordCount));
        CustomDimensions.Add('DurationCategory', GetDurationCategory(Duration));
        CustomDimensions.Add('ThroughputCategory', GetThroughputCategory(RecordCount, Duration));

        Session.LogMessage('PERF002', 'Batch operation completed', Verbosity::Normal,
                          DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                          CustomDimensions);
    end;

    local procedure GetRecordCountRange(RecordCount: Integer): Text
    begin
        case RecordCount of
            0..100: exit('1-100');
            101..1000: exit('101-1000');
            1001..10000: exit('1001-10000');
            else exit('10000+');
        end;
    end;

    local procedure GetDurationCategory(Duration: Duration): Text
    begin
        if Duration < 1000 then exit('Fast');
        if Duration < 5000 then exit('Normal');
        if Duration < 30000 then exit('Slow');
        exit('VerySlow');
    end;

    local procedure GetThroughputCategory(RecordCount: Integer; Duration: Duration): Text
    var
        RecordsPerSecond: Decimal;
    begin
        if Duration = 0 then exit('Instant');

        RecordsPerSecond := RecordCount / (Duration / 1000);

        if RecordsPerSecond > 1000 then exit('High');
        if RecordsPerSecond > 100 then exit('Medium');
        exit('Low');
    end;
}
```

## Best Practices Implementation

```al
// Comprehensive best practices example
codeunit 50306 "Telemetry Best Practices"
{
    procedure ImplementCorrectTelemetry()
    var
        CustomDimensions: Dictionary of [Text, Text];
        StartTime: DateTime;
        ProcessSuccess: Boolean;
    begin
        StartTime := CurrentDateTime;

        // ✅ CORRECT: Use meaningful event IDs and structured messages
        // ✅ CORRECT: Use appropriate verbosity levels
        // ✅ CORRECT: Use SystemMetadata classification for telemetry
        // ✅ CORRECT: Use ExtensionPublisher scope for app events

        CustomDimensions.Add('ProcessType', 'CustomerDataSync');      // Business context
        CustomDimensions.Add('UserCategory', GetUserCategory());      // Anonymized user info
        CustomDimensions.Add('CompanySize', GetCompanyCategory());    // Environmental context
        CustomDimensions.Add('FeatureVersion', '2.1.0');            // Version tracking

        Session.LogMessage('BP001', 'Customer data sync initiated', Verbosity::Normal,
                          DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                          CustomDimensions);

        // Execute process with proper error handling
        ProcessSuccess := TryExecuteSync();

        // Log completion with comprehensive context
        Clear(CustomDimensions);
        CustomDimensions.Add('ProcessType', 'CustomerDataSync');
        CustomDimensions.Add('Duration', GetDurationCategory(CurrentDateTime - StartTime));
        CustomDimensions.Add('Success', Format(ProcessSuccess));

        if ProcessSuccess then begin
            CustomDimensions.Add('Outcome', 'Success');
            Session.LogMessage('BP002', 'Customer data sync completed successfully', Verbosity::Normal,
                              DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                              CustomDimensions);
        end else begin
            CustomDimensions.Add('Outcome', 'Failed');
            CustomDimensions.Add('ErrorCategory', GetErrorCategory());
            Session.LogMessage('BP003', 'Customer data sync failed', Verbosity::Error,
                              DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher,
                              CustomDimensions);
        end;
    end;

    [TryFunction]
    local procedure TryExecuteSync()
    begin
        // Simulate sync operation that might fail
        Sleep(Random(5000));
        if Random(10) > 8 then
            Error('Simulated sync failure');
    end;

    local procedure GetUserCategory(): Text
    begin
        // Return user role category without identifying specific user
        exit('Administrator'); // Simplified - implement actual role detection
    end;

    local procedure GetCompanyCategory(): Text
    begin
        // Return company size category for environmental context
        exit('Medium'); // Simplified - implement actual size detection
    end;

    local procedure GetDurationCategory(Duration: Duration): Text
    begin
        if Duration < 2000 then exit('Fast');
        if Duration < 10000 then exit('Normal');
        exit('Slow');
    end;

    local procedure GetErrorCategory(): Text
    var
        ErrorText: Text;
    begin
        ErrorText := LowerCase(GetLastErrorText());

        if StrPos(ErrorText, 'permission') > 0 then exit('Permission');
        if StrPos(ErrorText, 'network') > 0 then exit('Network');
        if StrPos(ErrorText, 'timeout') > 0 then exit('Timeout');
        exit('General');
    end;
}
```

## Key Implementation Guidelines

### Event ID Strategy
- Use consistent prefixes for different functional areas
- Keep IDs unique and sequential within your extension
- Document ID meanings for long-term maintenance

### Custom Dimensions Best Practices
- Use descriptive, PascalCase key names
- Keep values concise but meaningful
- Limit dimensions per event (5-10 recommended)
- Use categorical values instead of exact numbers

### Privacy Compliance Checklist
- Never log customer names, addresses, or identifiable information
- Use hashing or anonymization for sensitive identifiers
- Categorize numerical data instead of exact values
- Review all telemetry for privacy violations before deployment

### Performance Considerations
- Avoid telemetry in tight loops
- Use sampling for high-frequency events
- Aggregate data for batch operations
- Monitor telemetry impact on application performance