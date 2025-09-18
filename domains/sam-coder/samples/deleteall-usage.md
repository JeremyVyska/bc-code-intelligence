# DeleteAll Method - Basic Usage Patterns

## Simple DeleteAll Implementation
```al
codeunit 50100 "Customer Data Cleanup"
{
    procedure CleanupInactiveCustomers(DaysInactive: Integer)
    var
        Customer: Record Customer;
        CutoffDate: Date;
    begin
        CutoffDate := CalcDate(StrSubstNo('-%1D', DaysInactive), Today());
        
        // Filter customers who haven't placed orders since cutoff date
        Customer.Reset();
        Customer.SetFilter("Last Date Modified", '<%1', CutoffDate);
        Customer.SetRange(Blocked, Customer.Blocked::" ");  // Only non-blocked customers
        
        // DeleteAll removes all matching records and executes OnDelete triggers
        if Customer.FindSet() then
            Customer.DeleteAll(true);  // RunTrigger = true (recommended)
    end;
}
```

## DeleteAll with Record Validation
```al
table 50100 "Temporary Sales Data"
{
    DataPerCompany = true;
    
    fields
    {
        field(1; "Entry No."; Integer) { AutoIncrement = true; }
        field(10; "Customer No."; Code[20]) { TableRelation = Customer; }
        field(20; "Sales Amount"; Decimal) { }
        field(30; "Process Date"; Date) { }
    }
    
    keys
    {
        key(PK; "Entry No.") { Clustered = true; }
        key(ProcessDate; "Process Date") { }
    }
    
    trigger OnDelete()
    begin
        // This trigger ALWAYS executes during DeleteAll operations
        // Cannot be bypassed - critical for data integrity
        if "Sales Amount" > 10000 then
            Error('Cannot delete high-value sales record: Entry %1', "Entry No.");
            
        // Log deletion for audit trail
        LogDeletion(Rec);
    end;
    
    local procedure LogDeletion(var TempSalesRec: Record "Temporary Sales Data")
    var
        AuditLog: Record "Change Log Entry";
    begin
        // Audit logging implementation
        // This ensures all deletions are tracked
    end;
}

codeunit 50101 "Sales Data Management"
{
    procedure CleanupProcessedData(ProcessDate: Date)
    var
        TempSalesData: Record "Temporary Sales Data";
    begin
        // Clean up processed sales data older than specified date
        TempSalesData.Reset();
        TempSalesData.SetFilter("Process Date", '<%1', ProcessDate);
        
        // DeleteAll will execute OnDelete trigger for each record
        // This ensures audit logging and validation occurs
        if not TempSalesData.IsEmpty() then begin
            Message('Cleaning up %1 processed sales records', TempSalesData.Count());
            TempSalesData.DeleteAll(true);  // Always use RunTrigger = true for safety
        end;
    end;
}
```

## Error Handling with DeleteAll
```al
codeunit 50102 "Order Cleanup Service"
{
    procedure DeleteCancelledOrders(var ErrorsOccurred: Boolean): Integer
    var
        SalesHeader: Record "Sales Header";
        DeleteCount: Integer;
        OriginalCount: Integer;
    begin
        ErrorsOccurred := false;
        
        // Find cancelled orders older than 30 days
        SalesHeader.Reset();
        SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
        SalesHeader.SetRange(Status, SalesHeader.Status::Released);
        SalesHeader.SetFilter("Order Date", '<%1', CalcDate('-30D', Today()));
        
        OriginalCount := SalesHeader.Count();
        
        // Attempt deletion with error handling
        if not SalesHeader.IsEmpty() then begin
            if not TryDeleteOrders(SalesHeader) then begin
                ErrorsOccurred := true;
                Error('Failed to delete cancelled orders: %1', GetLastErrorText());
            end;
            
            // Calculate how many were actually deleted
            SalesHeader.Reset();
            SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
            SalesHeader.SetRange(Status, SalesHeader.Status::Released);
            SalesHeader.SetFilter("Order Date", '<%1', CalcDate('-30D', Today()));
            DeleteCount := OriginalCount - SalesHeader.Count();
        end;
        
        exit(DeleteCount);
    end;
    
    [TryFunction]
    local procedure TryDeleteOrders(var SalesHeader: Record "Sales Header")
    begin
        // DeleteAll with error handling
        // OnDelete triggers will execute and may raise errors
        SalesHeader.DeleteAll(true);
    end;
}
```

## Best Practices Demonstrated

### 1. Always Use RunTrigger Parameter
```al
// CORRECT: Always specify RunTrigger parameter explicitly
Customer.DeleteAll(true);   // Recommended - ensures data integrity

// AVOID: Omitting parameter (defaults to false in some contexts)
Customer.DeleteAll();       // May skip important validation logic
```

### 2. Proper Filter Management
```al
// CORRECT: Clear filters before deletion
MyRecord.Reset();
MyRecord.SetRange("Status", MyRecord.Status::Processed);
MyRecord.DeleteAll(true);

// INCORRECT: Assuming current filters
// MyRecord.DeleteAll(true);  // May delete unexpected records
```

### 3. Transaction Safety
```al
procedure SafeBulkDeletion(var TempData: Record "Temporary Data")
begin
    // Large deletions should be performed in manageable chunks
    if TempData.Count() > 10000 then begin
        // Process in batches to avoid transaction timeout
        ProcessDeletionInBatches(TempData);
    end else begin
        // Small dataset - single operation is safe
        TempData.DeleteAll(true);
    end;
end;
```