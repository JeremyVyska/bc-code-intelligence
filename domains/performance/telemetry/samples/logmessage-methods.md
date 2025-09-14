# BC Session LogMessage API Usage Patterns - AL Code Samples

## Basic LogMessage Implementation
```al
codeunit 50100 "Telemetry Logger Examples"
{
    // Example 1: Basic information logging
    procedure LogBasicInformation()
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions.Add('Operation', 'CustomerCreation');
        CustomDimensions.Add('UserID', UserId());
        
        Session.LogMessage('0001', 'Customer creation initiated', Verbosity::Normal, 
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 2: Error logging with context
    procedure LogErrorWithContext(ErrorMessage: Text; SourceTable: Text; RecordID: Text)
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions.Add('ErrorSource', 'DataValidation');
        CustomDimensions.Add('TableName', SourceTable);
        CustomDimensions.Add('RecordID', RecordID);
        CustomDimensions.Add('CompanyName', CompanyName());
        
        Session.LogMessage('0002', ErrorMessage, Verbosity::Error, 
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 3: Performance timing logging
    procedure LogPerformanceMetrics(OperationName: Text; DurationMs: Integer; RecordCount: Integer)
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions.Add('Operation', OperationName);
        CustomDimensions.Add('DurationMs', Format(DurationMs));
        CustomDimensions.Add('RecordCount', Format(RecordCount));
        CustomDimensions.Add('Category', 'Performance');
        
        Session.LogMessage('0003', 'Performance metric captured', Verbosity::Normal,
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;
}
```

## Advanced LogMessage Patterns
```al
codeunit 50101 "Advanced Telemetry Patterns"
{
    // Example 4: Transaction-aware logging
    procedure LogTransactionOperation(TransactionID: Text; OperationType: Text; Status: Text)
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions.Add('TransactionID', TransactionID);
        CustomDimensions.Add('OperationType', OperationType);
        CustomDimensions.Add('Status', Status);
        CustomDimensions.Add('Timestamp', Format(CurrentDateTime, 0, '<Year4>-<Month,2>-<Day,2>T<Hours24>:<Minutes,2>:<Seconds,2>'));
        
        Session.LogMessage('0004', StrSubstNo('Transaction %1 %2: %3', OperationType, TransactionID, Status),
            Verbosity::Normal, DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 5: Conditional verbose logging
    procedure LogDetailedOperation(OperationDetails: Text; IsVerbose: Boolean)
    var
        CustomDimensions: Dictionary of [Text, Text];
        LogVerbosity: Verbosity;
    begin
        CustomDimensions.Add('DetailLevel', 'Extended');
        CustomDimensions.Add('Source', 'BusinessLogic');
        
        if IsVerbose then
            LogVerbosity := Verbosity::Verbose
        else
            LogVerbosity := Verbosity::Normal;
            
        Session.LogMessage('0005', OperationDetails, LogVerbosity,
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 6: Structured logging with multiple dimensions
    procedure LogStructuredEvent(EventName: Text; EventData: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        DataKey: Text;
    begin
        CustomDimensions.Add('EventName', EventName);
        CustomDimensions.Add('EventID', CreateGuid());
        
        // Merge additional event data into custom dimensions
        foreach DataKey in EventData.Keys do
            CustomDimensions.Add('Data_' + DataKey, EventData.Get(DataKey));
            
        Session.LogMessage('0006', StrSubstNo('Structured event: %1', EventName), Verbosity::Normal,
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;
}
```

## Integration with BC Events
```al
codeunit 50102 "Event-Driven Telemetry"
{
    // Example 7: Document posting telemetry
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", 'OnAfterPostSalesDoc', '', false, false)]
    local procedure OnAfterPostSalesDoc(var SalesHeader: Record "Sales Header"; var GenJnlPostLine: Codeunit "Gen. Jnl.-Post Line"; SalesShptHdrNo: Code[20])
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions.Add('DocumentType', Format(SalesHeader."Document Type"));
        CustomDimensions.Add('DocumentNo', SalesHeader."No.");
        CustomDimensions.Add('CustomerNo', SalesHeader."Sell-to Customer No.");
        CustomDimensions.Add('Amount', Format(SalesHeader."Amount Including VAT"));
        
        Session.LogMessage('0007', 'Sales document posted successfully', Verbosity::Normal,
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 8: Error event telemetry
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", 'OnBeforePostSalesDoc', '', false, false)]
    local procedure OnBeforePostSalesDoc(var SalesHeader: Record "Sales Header")
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions.Add('DocumentType', Format(SalesHeader."Document Type"));
        CustomDimensions.Add('DocumentNo', SalesHeader."No.");
        CustomDimensions.Add('ValidationStage', 'PrePosting');
        
        Session.LogMessage('0008', 'Sales document posting initiated', Verbosity::Verbose,
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;
}
```