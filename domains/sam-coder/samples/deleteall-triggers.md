# DeleteAll Method - OnDelete Trigger Integration

## Understanding Trigger Execution with DeleteAll
```al
table 50200 "Order Processing Log"
{
    fields
    {
        field(1; "Entry No."; Integer) { AutoIncrement = true; }
        field(10; "Order No."; Code[20]) { }
        field(20; "Customer No."; Code[20]) { TableRelation = Customer; }
        field(30; "Process Status"; Enum "Process Status") { }
        field(40; "Created DateTime"; DateTime) { }
    }
    
    keys
    {
        key(PK; "Entry No.") { Clustered = true; }
        key(OrderStatus; "Order No.", "Process Status") { }
    }
    
    trigger OnDelete()
    var
        RelatedOrderLine: Record "Order Line Processing";
        AuditEntry: Record "Deletion Audit Log";
    begin
        // CRITICAL: This trigger ALWAYS executes during DeleteAll
        // No way to bypass this - ensures data integrity is maintained
        
        // 1. Validate deletion is allowed
        if "Process Status" = "Process Status"::Active then
            Error('Cannot delete active processing log entry %1', "Entry No.");
        
        // 2. Clean up related records
        RelatedOrderLine.Reset();
        RelatedOrderLine.SetRange("Log Entry No.", "Entry No.");
        if not RelatedOrderLine.IsEmpty() then
            RelatedOrderLine.DeleteAll(true);  // Cascaded deletion with triggers
        
        // 3. Create audit trail
        AuditEntry.Init();
        AuditEntry."Table ID" := Database::"Order Processing Log";
        AuditEntry."Record ID" := RecordId();
        AuditEntry."Deletion Time" := CurrentDateTime;
        AuditEntry."User ID" := UserId;
        AuditEntry.Insert(true);
        
        // 4. Update related master data
        UpdateOrderStatistics("Order No.");
    end;
    
    local procedure UpdateOrderStatistics(OrderNo: Code[20])
    var
        OrderHeader: Record "Sales Header";
    begin
        // Update order processing statistics after log deletion
        if OrderHeader.Get(OrderHeader."Document Type"::Order, OrderNo) then begin
            OrderHeader."Processing Log Count" -= 1;
            OrderHeader.Modify(true);
        end;
    end;
}
```

## Key Principles

### 1. Triggers Always Execute
- DeleteAll with RunTrigger=true executes OnDelete for every record
- No bypass mechanism exists - ensures data integrity
- Plan trigger logic for bulk operation scenarios

### 2. Error Propagation
- Any trigger error stops entire DeleteAll operation
- Consider try-function patterns for graceful error handling
- Individual record processing may be needed for partial success scenarios