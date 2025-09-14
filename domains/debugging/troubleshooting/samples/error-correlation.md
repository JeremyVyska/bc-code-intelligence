# Error Correlation Implementation Patterns - AL Code Samples

## Correlation ID Management
```al
codeunit 50140 "Error Correlation Manager"
{
    var
        CurrentCorrelationId: Text;
        CorrelationIdStack: List of [Text];
        ErrorContext: Dictionary of [Text, Text];

    // Example 1: Transaction-scoped correlation tracking
    procedure BeginCorrelatedOperation(OperationName: Text): Text
    var
        CorrelationId: Text;
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CorrelationId := CreateCorrelationId();
        
        // Push current correlation ID to stack for nested operations
        if CurrentCorrelationId <> '' then
            CorrelationIdStack.Add(CurrentCorrelationId);
            
        CurrentCorrelationId := CorrelationId;
        
        // Initialize error context for this correlation
        ClearErrorContext();
        ErrorContext.Add('OperationName', OperationName);
        ErrorContext.Add('StartTime', Format(CurrentDateTime, 0, 9));
        ErrorContext.Add('UserID', UserId());
        ErrorContext.Add('CompanyName', CompanyName());
        
        // Log operation start
        CustomDimensions := BuildCorrelationDimensions(CorrelationId);
        CustomDimensions.Add('EventType', 'OperationStart');
        CustomDimensions.Add('OperationName', OperationName);
        
        Session.LogMessage('COR001', StrSubstNo('Operation started: %1', OperationName), 
            Verbosity::Verbose, DataClassification::SystemMetadata, 
            TelemetryScope::ExtensionPublisher, CustomDimensions);
            
        exit(CorrelationId);
    end;

    procedure EndCorrelatedOperation(CorrelationId: Text; Success: Boolean)
    var
        CustomDimensions: Dictionary of [Text, Text];
        Duration: Duration;
        StartTimeText: Text;
        StartTime: DateTime;
    begin
        CustomDimensions := BuildCorrelationDimensions(CorrelationId);
        CustomDimensions.Add('EventType', 'OperationEnd');
        CustomDimensions.Add('Success', Format(Success));
        
        // Calculate operation duration
        if ErrorContext.Get('StartTime', StartTimeText) then begin
            if Evaluate(StartTime, StartTimeText) then begin
                Duration := CurrentDateTime - StartTime;
                CustomDimensions.Add('DurationMs', Format(Duration));
            end;
        end;
        
        // Add operation context
        AddErrorContextToDimensions(CustomDimensions);
        
        Session.LogMessage('COR002', 
            StrSubstNo('Operation completed: %1 (Success: %2)', 
                ErrorContext.Get('OperationName'), Success), 
            Verbosity::Normal, DataClassification::SystemMetadata, 
            TelemetryScope::ExtensionPublisher, CustomDimensions);
            
        // Restore previous correlation ID
        if CorrelationIdStack.Count > 0 then begin
            CurrentCorrelationId := CorrelationIdStack.Get(CorrelationIdStack.Count);
            CorrelationIdStack.RemoveAt(CorrelationIdStack.Count);
        end else begin
            CurrentCorrelationId := '';
            ClearErrorContext();
        end;
    end;

    // Example 2: Error correlation with full context capture
    procedure LogCorrelatedError(ErrorMessage: Text; ErrorCode: Text; AdditionalContext: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        StackTrace: Text;
    begin
        if CurrentCorrelationId = '' then
            CurrentCorrelationId := CreateCorrelationId();
            
        CustomDimensions := BuildCorrelationDimensions(CurrentCorrelationId);
        CustomDimensions.Add('EventType', 'Error');
        CustomDimensions.Add('ErrorCode', ErrorCode);
        CustomDimensions.Add('ErrorMessage', ErrorMessage);
        
        // Add error context
        AddErrorContextToDimensions(CustomDimensions);
        
        // Add additional context provided by caller
        AddAdditionalContextToDimensions(CustomDimensions, AdditionalContext);
        
        // Capture stack trace information
        StackTrace := CaptureStackTrace();
        if StackTrace <> '' then
            CustomDimensions.Add('StackTrace', StackTrace);
            
        Session.LogMessage('COR003', StrSubstNo('Correlated error: %1', ErrorMessage),
            Verbosity::Error, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    local procedure CreateCorrelationId(): Text
    begin
        exit(CreateGuid());
    end;

    local procedure BuildCorrelationDimensions(CorrelationId: Text): Dictionary of [Text, Text]
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions.Add('CorrelationId', CorrelationId);
        CustomDimensions.Add('SessionId', Format(SessionId()));
        CustomDimensions.Add('UserID', UserId());
        CustomDimensions.Add('CompanyName', CompanyName());
        CustomDimensions.Add('Timestamp', Format(CurrentDateTime, 0, 9));
        exit(CustomDimensions);
    end;

    local procedure ClearErrorContext()
    begin
        ErrorContext.Clear();
    end;

    local procedure AddErrorContextToDimensions(var CustomDimensions: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in ErrorContext.Keys do
            CustomDimensions.Add('Context_' + Key, ErrorContext.Get(Key));
    end;

    local procedure AddAdditionalContextToDimensions(var CustomDimensions: Dictionary of [Text, Text]; AdditionalContext: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in AdditionalContext.Keys do
            CustomDimensions.Add(Key, AdditionalContext.Get(Key));
    end;

    local procedure CaptureStackTrace(): Text
    begin
        // Implement stack trace capture if available
        exit('');
    end;
}
```

## Cross-Process Error Correlation
```al
codeunit 50141 "Cross Process Correlation"
{
    // Example 3: API integration error correlation
    procedure LogAPICorrelatedCall(APIEndpoint: Text; HTTPMethod: Text; RequestCorrelationId: Text): Text
    var
        CallCorrelationId: Text;
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CallCorrelationId := CreateGuid();
        
        CustomDimensions.Add('CallCorrelationId', CallCorrelationId);
        CustomDimensions.Add('RequestCorrelationId', RequestCorrelationId);
        CustomDimensions.Add('APIEndpoint', APIEndpoint);
        CustomDimensions.Add('HTTPMethod', HTTPMethod);
        CustomDimensions.Add('EventType', 'APICallStart');
        CustomDimensions.Add('UserID', UserId());
        CustomDimensions.Add('CompanyName', CompanyName());
        
        Session.LogMessage('API001', StrSubstNo('API call initiated: %1 %2', HTTPMethod, APIEndpoint),
            Verbosity::Normal, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
            
        exit(CallCorrelationId);
    end;

    procedure LogAPICorrelatedResponse(CallCorrelationId: Text; HTTPStatusCode: Integer; ResponseTime: Integer; ErrorMessage: Text)
    var
        CustomDimensions: Dictionary of [Text, Text];
        IsSuccess: Boolean;
    begin
        IsSuccess := (HTTPStatusCode >= 200) and (HTTPStatusCode < 300);
        
        CustomDimensions.Add('CallCorrelationId', CallCorrelationId);
        CustomDimensions.Add('HTTPStatusCode', Format(HTTPStatusCode));
        CustomDimensions.Add('ResponseTime', Format(ResponseTime));
        CustomDimensions.Add('Success', Format(IsSuccess));
        CustomDimensions.Add('EventType', 'APICallEnd');
        
        if not IsSuccess then begin
            CustomDimensions.Add('ErrorMessage', ErrorMessage);
            CustomDimensions.Add('ErrorCategory', 'APIError');
        end;
        
        Session.LogMessage('API002', 
            StrSubstNo('API call completed: Status %1, Duration %2ms', HTTPStatusCode, ResponseTime),
            if IsSuccess then Verbosity::Normal else Verbosity::Error,
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 4: Business process correlation
    procedure TrackBusinessProcessFlow(ProcessName: Text; ProcessStage: Text; BusinessData: Dictionary of [Text, Text])
    var
        ProcessCorrelationId: Text;
        CustomDimensions: Dictionary of [Text, Text];
    begin
        ProcessCorrelationId := GetOrCreateProcessCorrelation(ProcessName, BusinessData);
        
        CustomDimensions.Add('ProcessCorrelationId', ProcessCorrelationId);
        CustomDimensions.Add('ProcessName', ProcessName);
        CustomDimensions.Add('ProcessStage', ProcessStage);
        CustomDimensions.Add('EventType', 'ProcessFlow');
        
        // Add business context
        AddBusinessContextToDimensions(CustomDimensions, BusinessData);
        
        Session.LogMessage('PROC001', StrSubstNo('Business process flow: %1 - %2', ProcessName, ProcessStage),
            Verbosity::Normal, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    local procedure GetOrCreateProcessCorrelation(ProcessName: Text; BusinessData: Dictionary of [Text, Text]): Text
    var
        ProcessKey: Text;
        ProcessCorrelationMap: Dictionary of [Text, Text];
    begin
        ProcessKey := ProcessName + '_' + BusinessData.Get('DocumentNo');
        
        if not ProcessCorrelationMap.ContainsKey(ProcessKey) then
            ProcessCorrelationMap.Add(ProcessKey, CreateGuid());
            
        exit(ProcessCorrelationMap.Get(ProcessKey));
    end;

    local procedure AddBusinessContextToDimensions(var CustomDimensions: Dictionary of [Text, Text]; BusinessData: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in BusinessData.Keys do
            CustomDimensions.Add('Business_' + Key, BusinessData.Get(Key));
    end;
}
```

## Database Transaction Correlation
```al
codeunit 50142 "Database Transaction Correlation"
{
    var
        TransactionStartTimes: Dictionary of [Text, DateTime];

    // Example 5: Database transaction correlation
    procedure BeginDatabaseTransaction(TransactionType: Text; TableName: Text): Text
    var
        TransactionId: Text;
        CustomDimensions: Dictionary of [Text, Text];
    begin
        TransactionId := CreateTransactionId();
        
        TransactionStartTimes.Add(TransactionId, CurrentDateTime);
        
        CustomDimensions.Add('TransactionId', TransactionId);
        CustomDimensions.Add('TransactionType', TransactionType);
        CustomDimensions.Add('TableName', TableName);
        CustomDimensions.Add('EventType', 'TransactionStart');
        CustomDimensions.Add('UserID', UserId());
        CustomDimensions.Add('CompanyName', CompanyName());
        
        Session.LogMessage('TXN001', StrSubstNo('Database transaction started: %1 on %2', TransactionType, TableName),
            Verbosity::Verbose, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
            
        exit(TransactionId);
    end;

    procedure LogTransactionError(TransactionId: Text; ErrorMessage: Text; RecordContext: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        TransactionDuration: Duration;
        StartTime: DateTime;
    begin
        CustomDimensions.Add('TransactionId', TransactionId);
        CustomDimensions.Add('EventType', 'TransactionError');
        CustomDimensions.Add('ErrorMessage', ErrorMessage);
        CustomDimensions.Add('ErrorCategory', 'DatabaseError');
        
        // Calculate transaction duration
        if TransactionStartTimes.Get(TransactionId, StartTime) then begin
            TransactionDuration := CurrentDateTime - StartTime;
            CustomDimensions.Add('TransactionDuration', Format(TransactionDuration));
        end;
        
        // Add record context for debugging
        AddRecordContextToDimensions(CustomDimensions, RecordContext);
        
        Session.LogMessage('TXN002', StrSubstNo('Database transaction error: %1', ErrorMessage),
            Verbosity::Error, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    local procedure CreateTransactionId(): Text
    begin
        exit('TXN-' + Format(CurrentDateTime, 0, '<Year4><Month,2><Day,2><Hours24><Minutes,2><Seconds,2>') + 
             '-' + Format(Random(9999), 4, '<Integer,4><Filler Character,0>'));
    end;

    local procedure AddRecordContextToDimensions(var CustomDimensions: Dictionary of [Text, Text]; RecordContext: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in RecordContext.Keys do
            CustomDimensions.Add('Record_' + Key, RecordContext.Get(Key));
    end;
}
```