# Event Subscriber Architecture - AL Code Examples

## Good Examples

### Single Responsibility Subscriber
```al
codeunit 50100 "Customer Credit Check Subscriber"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", 'OnBeforePostSalesDoc', '', false, false)]
    local procedure OnBeforePostSalesDoc(var SalesHeader: Record "Sales Header"; CommitIsSupressed: Boolean; PreviewMode: Boolean; var HideProgressWindow: Boolean; var IsHandled: Boolean)
    var
        CreditManagement: Codeunit "Credit Management";
    begin
        // Single responsibility: Credit validation only
        if SalesHeader."Document Type" <> SalesHeader."Document Type"::Order then
            exit;

        if PreviewMode then
            exit;

        // Early exit if already handled
        if IsHandled then
            exit;

        // Focused credit check logic
        if not CreditManagement.ValidateCustomerCredit(SalesHeader."Sell-to Customer No.", SalesHeader."Amount Including VAT") then begin
            Error('Credit limit exceeded for customer %1', SalesHeader."Sell-to Customer No.");
        end;
    end;
}
```

### Error-Isolated Integration Subscriber
```al
codeunit 50101 "External System Integration"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Item Jnl.-Post Line", 'OnAfterPostItemJnlLine', '', false, false)]
    local procedure OnAfterPostItemJnlLine(var ItemJournalLine: Record "Item Journal Line"; ItemLedgerEntry: Record "Item Ledger Entry"; var ValueEntryNo: Integer; var InventoryPostingToGL: Codeunit "Inventory Posting To G/L")
    var
        ExternalSystemMgt: Codeunit "External System Management";
        IntegrationSetup: Record "Integration Setup";
    begin
        // Error isolation: Don't affect posting if integration fails
        if not IntegrationSetup.Get() then
            exit;

        if not IntegrationSetup."Enable Inventory Sync" then
            exit;

        // Only sync specific entry types
        if not (ItemJournalLine."Entry Type" in [ItemJournalLine."Entry Type"::Sale, ItemJournalLine."Entry Type"::Purchase]) then
            exit;

        // Isolated external call with error handling
        if not ExternalSystemMgt.TrySyncInventoryChange(ItemLedgerEntry) then begin
            // Log error but don't fail the posting
            LogIntegrationError('Inventory sync failed', ItemLedgerEntry."Entry No.");
        end;
    end;
}
```

### Performance-Conscious Bulk Processing
```al
codeunit 50102 "Audit Trail Subscriber"
{
    var
        TempAuditBuffer: Record "Audit Buffer" temporary;
        BufferSize: Integer;

    [EventSubscriber(ObjectType::Table, Database::"Sales Header", 'OnAfterModifyEvent', '', false, false)]
    local procedure OnAfterModifySalesHeader(var Rec: Record "Sales Header"; var xRec: Record "Sales Header"; RunTrigger: Boolean)
    begin
        // Performance: Early filtering
        if not RunTrigger then
            exit;

        if not AuditSetup.IsAuditEnabled(Database::"Sales Header") then
            exit;

        // Buffer changes for batch processing
        BufferAuditEntry(Rec, xRec);

        // Batch processing when buffer reaches threshold
        if TempAuditBuffer.Count >= BufferSize then
            ProcessAuditBuffer();
    end;

    local procedure BufferAuditEntry(var CurrentRecord: Record "Sales Header"; var PreviousRecord: Record "Sales Header")
    begin
        TempAuditBuffer.Init();
        TempAuditBuffer."Entry No." := TempAuditBuffer.Count + 1;
        TempAuditBuffer."Table No." := Database::"Sales Header";
        TempAuditBuffer."Record ID" := CurrentRecord.RecordId;
        TempAuditBuffer."Change DateTime" := CurrentDateTime;
        TempAuditBuffer.Insert();
    end;

    local procedure ProcessAuditBuffer()
    var
        AuditEntry: Record "Audit Entry";
    begin
        if TempAuditBuffer.IsEmpty then
            exit;

        // Batch insert audit entries
        TempAuditBuffer.FindSet();
        repeat
            AuditEntry.TransferFields(TempAuditBuffer);
            AuditEntry.Insert();
        until TempAuditBuffer.Next() = 0;

        // Clear buffer after processing
        TempAuditBuffer.DeleteAll();
    end;
}
```

### Background Session Integration
```al
codeunit 50103 "Document Workflow Subscriber"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Release Sales Document", 'OnAfterReleaseSalesDoc', '', false, false)]
    local procedure OnAfterReleaseSalesDoc(var SalesHeader: Record "Sales Header"; PreviewMode: Boolean; var LinesWereModified: Boolean)
    var
        WorkflowParameters: Record "Workflow Parameters" temporary;
    begin
        if PreviewMode then
            exit;

        // Only process specific document types
        if not (SalesHeader."Document Type" in [SalesHeader."Document Type"::Order, SalesHeader."Document Type"::Quote]) then
            exit;

        // Prepare parameters for background processing
        WorkflowParameters.Init();
        WorkflowParameters."Document Type" := SalesHeader."Document Type".AsInteger();
        WorkflowParameters."Document No." := SalesHeader."No.";
        WorkflowParameters."Customer No." := SalesHeader."Sell-to Customer No.";
        WorkflowParameters.Insert();

        // Execute workflow in background session to avoid blocking
        StartBackgroundWorkflow(WorkflowParameters);
    end;

    local procedure StartBackgroundWorkflow(var Parameters: Record "Workflow Parameters")
    begin
        // Background session execution prevents performance impact
        TaskScheduler.CreateTask(
            Codeunit::"Workflow Background Processor",
            Codeunit::"Workflow Error Handler",
            true,
            CompanyName,
            CurrentDateTime + 1000, // Delay 1 second
            Parameters.RecordId);
    end;
}
```

## Bad Examples

### Tightly Coupled Subscriber
```al
codeunit 50200 "Bad Integration Subscriber"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", 'OnBeforePostSalesDoc', '', false, false)]
    local procedure OnBeforePostSalesDoc(var SalesHeader: Record "Sales Header"; CommitIsSupressed: Boolean; PreviewMode: Boolean; var HideProgressWindow: Boolean; var IsHandled: Boolean)
    var
        SalesLine: Record "Sales Line";
        Customer: Record Customer;
        Item: Record Item;
    begin
        // BAD: Direct access to publisher's internal data
        if not Customer.Get(SalesHeader."Sell-to Customer No.") then begin
            // BAD: Bypassing publisher's validation logic
            SalesHeader."Sell-to Customer No." := '';
            SalesHeader.Modify();
        end;

        // BAD: Complex business logic in subscriber
        SalesLine.SetRange("Document Type", SalesHeader."Document Type");
        SalesLine.SetRange("Document No.", SalesHeader."No.");
        if SalesLine.FindSet() then begin
            repeat
                // BAD: Modifying publisher data without clear contract
                if Item.Get(SalesLine."No.") then begin
                    if Item."Unit Cost" = 0 then begin
                        SalesLine."Unit Cost" := 100; // BAD: Hardcoded business logic
                        SalesLine.Modify();
                    end;
                end;
            until SalesLine.Next() = 0;
        end;

        // BAD: No error handling - failures will break posting
    end;
}
```

### Performance-Degrading Subscriber
```al
codeunit 50201 "Slow Integration Subscriber"
{
    [EventSubscriber(ObjectType::Table, Database::"Item Ledger Entry", 'OnAfterInsertEvent', '', false, false)]
    local procedure OnAfterInsertItemLedgerEntry(var Rec: Record "Item Ledger Entry"; RunTrigger: Boolean)
    var
        Item: Record Item;
        Customer: Record Customer;
        Vendor: Record Vendor;
        ExternalAPI: Codeunit "External API Client";
    begin
        // BAD: No filtering - executes for every entry
        // BAD: Synchronous external API call slows down posting
        ExternalAPI.SendInventoryUpdate(Rec."Item No.", Rec.Quantity);

        // BAD: Unnecessary database lookups on every insert
        if Item.Get(Rec."Item No.") then begin
            if Customer.Get(Item."Vendor No.") then begin // BAD: Wrong relationship
                // BAD: More expensive operations
                ExternalAPI.UpdateCustomerInventory(Customer."No.", Rec.Quantity);
            end;
        end;

        // BAD: Complex calculations in high-frequency event
        CalculateComplexStatistics(Rec);
    end;

    local procedure CalculateComplexStatistics(ItemLedgerEntry: Record "Item Ledger Entry")
    var
        AllEntries: Record "Item Ledger Entry";
    begin
        // BAD: Expensive calculation on every insert
        AllEntries.SetRange("Item No.", ItemLedgerEntry."Item No.");
        AllEntries.CalcSums(Quantity, "Cost Amount (Actual)");
        // Complex processing...
    end;
}
```

### Error-Propagating Subscriber
```al
codeunit 50202 "Unreliable Subscriber"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", 'OnAfterPostSalesDoc', '', false, false)]
    local procedure OnAfterPostSalesDoc(var SalesHeader: Record "Sales Header"; var SalesInvoiceHeader: Record "Sales Invoice Header"; var SalesCrMemoHeader: Record "Sales Cr.Memo Header"; CommitIsSupressed: Boolean)
    var
        ExternalSystem: Codeunit "Unreliable External System";
        NotificationEmail: Codeunit "Email Management";
    begin
        // BAD: No error handling - any failure stops processing
        ExternalSystem.PostDocumentToERP(SalesHeader);

        // BAD: Dependent on external system reliability
        ExternalSystem.UpdateCustomerAccount(SalesHeader."Sell-to Customer No.");

        // BAD: Multiple failure points without isolation
        NotificationEmail.SendConfirmation(SalesHeader."Sell-to Customer No.");

        // BAD: No fallback or retry mechanism
        ExternalSystem.SyncPaymentTerms(SalesHeader."Payment Terms Code");
    end;
}
```

## Best Practices

### Conditional Processing Optimization
```al
codeunit 50300 "Optimized Event Subscriber"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Item Jnl.-Post Line", 'OnBeforePostItemJnlLine', '', false, false)]
    local procedure OnBeforePostItemJnlLine(var ItemJournalLine: Record "Item Journal Line"; var IsHandled: Boolean)
    var
        IntegrationSetup: Record "Integration Setup";
    begin
        // Early exit conditions for performance
        if IsHandled then
            exit;

        if ItemJournalLine."Entry Type" <> ItemJournalLine."Entry Type"::Sale then
            exit;

        if not IntegrationSetup.Get() then
            exit;

        if not IntegrationSetup."Validate Inventory on Sale" then
            exit;

        // Focused processing only when needed
        ValidateInventoryAvailability(ItemJournalLine);
    end;
}
```

### Circuit Breaker Pattern
```al
codeunit 50301 "Resilient Integration Subscriber"
{
    var
        FailureCount: Integer;
        LastFailureTime: DateTime;
        CircuitBreakerOpen: Boolean;

    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", 'OnAfterPostSalesDoc', '', false, false)]
    local procedure OnAfterPostSalesDoc(var SalesHeader: Record "Sales Header"; var SalesInvoiceHeader: Record "Sales Invoice Header"; var SalesCrMemoHeader: Record "Sales Cr.Memo Header"; CommitIsSupressed: Boolean)
    begin
        // Circuit breaker prevents cascading failures
        if CircuitBreakerOpen then begin
            if (CurrentDateTime - LastFailureTime) > 300000 then // 5 minutes
                ResetCircuitBreaker()
            else
                exit;
        end;

        // Attempt integration with failure tracking
        if not TryIntegrateDocument(SalesHeader) then
            HandleIntegrationFailure();
    end;

    local procedure TryIntegrateDocument(SalesHeader: Record "Sales Header"): Boolean
    var
        ExternalAPI: Codeunit "External API Client";
    begin
        exit(ExternalAPI.TryPostDocument(SalesHeader));
    end;

    local procedure HandleIntegrationFailure()
    begin
        FailureCount += 1;
        LastFailureTime := CurrentDateTime;

        // Open circuit breaker after 3 consecutive failures
        if FailureCount >= 3 then
            CircuitBreakerOpen := true;

        // Log failure for monitoring
        LogIntegrationFailure();
    end;

    local procedure ResetCircuitBreaker()
    begin
        FailureCount := 0;
        CircuitBreakerOpen := false;
        LogCircuitBreakerReset();
    end;
}
```

### Versioned Event Contract
```al
codeunit 50302 "Versioned Event Subscriber"
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Custom Publisher", 'OnBusinessEvent', '', false, false)]
    local procedure OnBusinessEvent(EventData: Text; EventVersion: Integer)
    var
        EventProcessor: Codeunit "Event Data Processor";
    begin
        // Handle different event contract versions
        case EventVersion of
            1:
                ProcessEventV1(EventData);
            2:
                ProcessEventV2(EventData);
            3:
                ProcessEventV3(EventData);
            else
                LogUnsupportedEventVersion(EventVersion);
        end;
    end;

    local procedure ProcessEventV1(EventData: Text)
    begin
        // Legacy event format handling
    end;

    local procedure ProcessEventV2(EventData: Text)
    begin
        // Enhanced event format with backward compatibility
    end;

    local procedure ProcessEventV3(EventData: Text)
    begin
        // Latest event format
    end;
}
```

## Architecture Guidelines

1. **Maintain loose coupling** - Depend only on event parameters, not publisher internals
2. **Implement error isolation** - Subscriber failures shouldn't break core functionality
3. **Optimize for performance** - Use early exits and efficient processing
4. **Design for reliability** - Include circuit breakers and retry mechanisms
5. **Plan for evolution** - Support event contract versioning
6. **Monitor and log** - Implement comprehensive logging for troubleshooting