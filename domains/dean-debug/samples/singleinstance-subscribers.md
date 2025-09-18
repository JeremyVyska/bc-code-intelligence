# SingleInstance Subscriber Patterns - AL Code Samples

## Basic SingleInstance Implementation
```al
// SingleInstance subscriber codeunit - only one instance exists per session
[SingleInstance]
codeunit 50100 "Sales Order Workflow Manager"
{
    var
        WorkflowActive: Boolean;
        ProcessedOrderCount: Integer;
        LastProcessedTime: DateTime;

    // Event subscriber that benefits from SingleInstance pattern
    [EventSubscriber(ObjectType::Table, Database::"Sales Header", 'OnAfterInsertEvent', '', false, false)]
    local procedure OnSalesOrderInsert(var Rec: Record "Sales Header")
    begin
        // SingleInstance ensures this counter is accurate across all sessions
        if Rec."Document Type" = Rec."Document Type"::Order then begin
            ProcessedOrderCount += 1;
            LastProcessedTime := CurrentDateTime;
            
            // State persists because of SingleInstance
            if ProcessedOrderCount mod 100 = 0 then
                SendBatchNotification(ProcessedOrderCount);
        end;
    end;

    // Method that leverages persistent state
    procedure GetProcessingStats(var OrderCount: Integer; var LastProcessed: DateTime)
    begin
        OrderCount := ProcessedOrderCount;
        LastProcessed := LastProcessedTime;
    end;

    local procedure SendBatchNotification(Count: Integer)
    begin
        // Implementation for milestone notifications
        Message('Processed %1 sales orders in this session', Count);
    end;
}
```

## Advanced SingleInstance with Configuration Management
```al
// SingleInstance subscriber for system-wide configuration caching
[SingleInstance]
codeunit 50101 "System Configuration Manager"
{
    var
        ConfigurationLoaded: Boolean;
        CacheValidUntil: DateTime;
        SystemSettings: Dictionary of [Text, Text];

    // Expensive configuration loading done once per server instance
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Company-Initialize", 'OnCompanyInitialize', '', false, false)]
    local procedure OnCompanyInitialize()
    begin
        LoadSystemConfiguration();
    end;

    // Public method that benefits from cached configuration
    procedure GetConfigurationValue(SettingKey: Text): Text
    var
        SettingValue: Text;
    begin
        // Refresh cache if expired
        if (CacheValidUntil < CurrentDateTime) or (not ConfigurationLoaded) then
            LoadSystemConfiguration();

        if SystemSettings.Get(SettingKey, SettingValue) then
            exit(SettingValue)
        else
            exit('');
    end;

    local procedure LoadSystemConfiguration()
    begin
        Clear(SystemSettings);
        
        // Load configuration once - expensive operation
        // Simulate configuration loading
        SystemSettings.Set('MaxOrderValue', '100000');
        SystemSettings.Set('DefaultShipping', 'STANDARD');
        SystemSettings.Set('AutoApprovalLimit', '5000');

        ConfigurationLoaded := true;
        CacheValidUntil := CurrentDateTime + (60 * 60 * 1000); // 1 hour cache
    end;
}
```

## Performance Impact Demonstration
```al
// GOOD: SingleInstance subscriber for shared state
[SingleInstance]
codeunit 50102 "Performance Optimized Subscriber"
{
    var
        ExpensiveResourceHandle: Integer;
        ResourceInitialized: Boolean;

    [EventSubscriber(ObjectType::Table, Database::"Item Ledger Entry", 'OnAfterInsertEvent', '', false, false)]
    local procedure OnItemLedgerEntryInsert(var Rec: Record "Item Ledger Entry")
    begin
        // Resource initialized once per server instance
        if not ResourceInitialized then
            InitializeExpensiveResource();

        // Fast processing using pre-initialized resource
        ProcessItemTransaction(Rec, ExpensiveResourceHandle);
    end;

    local procedure InitializeExpensiveResource()
    begin
        // Expensive initialization (database connections, external API setup, etc.)
        Sleep(1000); // Simulating expensive setup
        ExpensiveResourceHandle := 12345;
        ResourceInitialized := true;
    end;

    local procedure ProcessItemTransaction(ItemLedger: Record "Item Ledger Entry"; ResourceHandle: Integer)
    begin
        // Fast processing using initialized resource
        // Implementation details...
    end;
}

// BAD: Normal subscriber recreated for each event (anti-pattern)
codeunit 50103 "Performance Poor Subscriber"
{
    [EventSubscriber(ObjectType::Table, Database::"Item Ledger Entry", 'OnAfterInsertEvent', '', false, false)]
    local procedure OnItemLedgerEntryInsert(var Rec: Record "Item Ledger Entry")
    var
        ExpensiveResourceHandle: Integer;
    begin
        // BAD: Expensive initialization happens for EVERY event
        InitializeExpensiveResource(ExpensiveResourceHandle);
        ProcessItemTransaction(Rec, ExpensiveResourceHandle);
    end;

    local procedure InitializeExpensiveResource(var ResourceHandle: Integer)
    begin
        // This runs for every single item ledger entry - very inefficient!
        Sleep(1000);
        ResourceHandle := 12345;
    end;

    local procedure ProcessItemTransaction(ItemLedger: Record "Item Ledger Entry"; ResourceHandle: Integer)
    begin
        // Processing happens after expensive setup each time
    end;
}
```

## Memory Management Best Practices
```al
// Comprehensive SingleInstance subscriber with proper lifecycle management
[SingleInstance]
codeunit 50104 "Enterprise Event Manager"
{
    var
        SubscriberActive: Boolean;
        EventProcessingQueue: List of [Text];
        MaxQueueSize: Integer;
        
    trigger OnRun()
    begin
        Initialize();
    end;

    local procedure Initialize()
    begin
        SubscriberActive := true;
        MaxQueueSize := 1000;
        Clear(EventProcessingQueue);
    end;

    [EventSubscriber(ObjectType::Table, Database::"Sales Header", 'OnAfterModifyEvent', '', false, false)]
    local procedure OnSalesHeaderModify(var Rec: Record "Sales Header")
    begin
        if not SubscriberActive then exit;
        
        QueueEvent('SalesHeaderModified', Rec."No.");
        ProcessEventQueue();
    end;

    // Queue management with SingleInstance persistence
    local procedure QueueEvent(EventType: Text; DocumentNo: Text)
    var
        EventDescription: Text;
    begin
        EventDescription := StrSubstNo('%1:%2:%3', EventType, DocumentNo, Format(CurrentDateTime));
        
        if EventProcessingQueue.Count >= MaxQueueSize then
            EventProcessingQueue.RemoveAt(1); // Remove oldest event
            
        EventProcessingQueue.Add(EventDescription);
    end;

    local procedure ProcessEventQueue()
    begin
        // Batch processing of queued events
        if EventProcessingQueue.Count >= 10 then begin
            ProcessBatchEvents();
            EventProcessingQueue.RemoveRange(1, 10);
        end;
    end;

    local procedure ProcessBatchEvents()
    var
        EventItem: Text;
        i: Integer;
    begin
        // Process events in batches for efficiency
        for i := 1 to MinValue(10, EventProcessingQueue.Count) do begin
            EventItem := EventProcessingQueue.Get(i);
            // Process individual event
        end;
    end;

    // Public method to control subscriber behavior
    procedure SetActiveState(Active: Boolean)
    begin
        SubscriberActive := Active;
        if not Active then
            Clear(EventProcessingQueue);
    end;

    local procedure MinValue(Value1: Integer; Value2: Integer): Integer
    begin
        if Value1 < Value2 then
            exit(Value1)
        else
            exit(Value2);
    end;
}
```