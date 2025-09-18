# DeleteAll SQL Translation and Performance - Examples

## Performance Comparison: DeleteAll vs Loop

```al
// High-Performance Approach: DeleteAll for bulk operations
// Translates to single SQL DELETE statement
codeunit 50200 "Order Cleanup Manager"
{
    procedure CleanupOldOrders(CutoffDate: Date): Integer
    var
        SalesHeader: Record "Sales Header";
        DeletedCount: Integer;
    begin
        // Single SQL operation - optimal for large datasets
        SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Quote);
        SalesHeader.SetFilter("Document Date", '<%1', CutoffDate);
        SalesHeader.SetRange(Status, SalesHeader.Status::Open);

        // Count before deletion for reporting
        DeletedCount := SalesHeader.Count();

        // Execute as single SQL DELETE statement
        // SQL: DELETE FROM "Sales Header" WHERE "Document Type" = 1
        //      AND "Document Date" < @CutoffDate AND Status = 0
        SalesHeader.DeleteAll();

        exit(DeletedCount);
    end;

    procedure CleanupByStatus(DocumentType: Enum "Sales Document Type"; StatusFilter: Text)
    var
        SalesHeader: Record "Sales Header";
    begin
        SalesHeader.SetRange("Document Type", DocumentType);
        SalesHeader.SetFilter(Status, StatusFilter);

        // Leverages database indexing for optimal performance
        // SQL engine handles WHERE clause optimization
        SalesHeader.DeleteAll();
    end;
}
```

## Event Bypass Behavior

```al
// Understanding Event Bypass: DeleteAll skips AL table events
codeunit 50201 "Archive vs Delete Demo"
{
    procedure SafeBulkDeletion(var SalesHeader: Record "Sales Header")
    var
        TempSalesHeader: Record "Sales Header" temporary;
        ArchiveManagement: Codeunit ArchiveManagement;
    begin
        // Collect records for event processing if needed
        if SalesHeader.FindSet() then
            repeat
                TempSalesHeader := SalesHeader;
                TempSalesHeader.Insert();
            until SalesHeader.Next() = 0;

        // Process AL events manually if required
        if TempSalesHeader.FindSet() then
            repeat
                // Execute necessary validations/archiving before bulk delete
                ArchiveManagement.StoreSalesDocument(TempSalesHeader, false);
            until TempSalesHeader.Next() = 0;

        // Now safe to use DeleteAll - events already processed
        SalesHeader.DeleteAll();
    end;

    procedure CompareEventBehavior()
    var
        SalesLine: Record "Sales Line";
        SingleLine: Record "Sales Line";
    begin
        // DELETEALL APPROACH: Events bypassed
        SalesLine.SetRange("Document No.", 'ORDER001');
        SalesLine.DeleteAll(); // OnDelete triggers NOT called

        // ITERATIVE APPROACH: Events execute
        SingleLine.SetRange("Document No.", 'ORDER002');
        if SingleLine.FindSet() then
            repeat
                SingleLine.Delete(); // OnDelete triggers ARE called
            until SingleLine.Next() = 0;
    end;
}
```

## SQL Translation Patterns

```al
// Filter optimization impacts SQL DELETE performance
codeunit 50202 "SQL Optimization Examples"
{
    procedure OptimizedFilters()
    var
        ItemLedgerEntry: Record "Item Ledger Entry";
    begin
        // GOOD: Uses indexed fields for optimal SQL performance
        ItemLedgerEntry.SetRange("Item No.", 'ITEM001');
        ItemLedgerEntry.SetRange("Posting Date", 20240101D, 20241231D);
        ItemLedgerEntry.SetRange("Entry Type", ItemLedgerEntry."Entry Type"::Sale);

        // SQL: DELETE FROM "Item Ledger Entry"
        //      WHERE "Item No." = 'ITEM001'
        //      AND "Posting Date" BETWEEN '2024-01-01' AND '2024-12-31'
        //      AND "Entry Type" = 1
        // Database can use compound index efficiently
        ItemLedgerEntry.DeleteAll();
    end;

    procedure SuboptimalFilters()
    var
        ItemLedgerEntry: Record "Item Ledger Entry";
    begin
        // SUBOPTIMAL: Complex filters may require full table scan
        ItemLedgerEntry.SetFilter("Description", '@*SAMPLE*');
        ItemLedgerEntry.SetFilter("Quantity", '>0&<100');

        // SQL: DELETE FROM "Item Ledger Entry"
        //      WHERE "Description" LIKE '%SAMPLE%'
        //      AND "Quantity" > 0 AND "Quantity" < 100
        // May require table scan without proper indexing
        ItemLedgerEntry.DeleteAll();
    end;
}
```

## Transaction Boundary Considerations

```al
// Managing large DeleteAll operations within transaction limits
codeunit 50203 "Batch Delete Manager"
{
    procedure LargeBatchDeletion(TableID: Integer; FilterText: Text): Boolean
    var
        RecRef: RecordRef;
        TotalRecords: Integer;
        BatchSize: Integer;
        ProcessedCount: Integer;
    begin
        RecRef.Open(TableID);
        RecRef.SetView(FilterText);

        TotalRecords := RecRef.Count();
        BatchSize := 10000; // Adjust based on transaction log limits

        if TotalRecords <= BatchSize then begin
            // Single DeleteAll for smaller datasets
            RecRef.DeleteAll();
            exit(true);
        end;

        // Process in batches for very large datasets
        repeat
            RecRef.SetView(FilterText);
            if RecRef.FindSet() then begin
                ProcessedCount := 0;
                repeat
                    RecRef.Delete();
                    ProcessedCount += 1;
                    if ProcessedCount >= BatchSize then begin
                        Commit(); // Release transaction log space
                        RecRef.SetView(FilterText);
                        RecRef.FindFirst();
                        ProcessedCount := 0;
                    end;
                until (RecRef.Next() = 0) or (ProcessedCount >= BatchSize);
            end;
        until RecRef.Count() = 0;

        exit(true);
    end;
}
```

## Performance Monitoring

```al
// Measuring DeleteAll performance characteristics
codeunit 50204 "Delete Performance Monitor"
{
    procedure BenchmarkDeletionMethods(RecordCount: Integer)
    var
        TempInteger: Record "Integer" temporary;
        StartTime: Time;
        DeleteAllTime: Duration;
        IterativeTime: Duration;
        i: Integer;
    begin
        // Setup test data
        for i := 1 to RecordCount do begin
            TempInteger.Number := i;
            TempInteger.Insert();
        end;

        // Benchmark DeleteAll approach
        StartTime := Time;
        TempInteger.DeleteAll();
        DeleteAllTime := Time - StartTime;

        // Recreate test data for iterative test
        for i := 1 to RecordCount do begin
            TempInteger.Number := i;
            TempInteger.Insert();
        end;

        // Benchmark iterative deletion
        StartTime := Time;
        if TempInteger.FindSet() then
            repeat
                TempInteger.Delete();
            until TempInteger.Next() = 0;
        IterativeTime := Time - StartTime;

        // Log performance comparison
        Message('DeleteAll: %1ms, Iterative: %2ms, Ratio: %3:1',
                DeleteAllTime, IterativeTime,
                Round(IterativeTime / DeleteAllTime, 0.1));
    end;
}
```

## Anti-Pattern: Mixed Deletion Approaches

```al
// ANTI-PATTERN: Inconsistent deletion methods in same operation
codeunit 50299 "Deletion Anti-Patterns" // DON'T DO THIS
{
    procedure InconsistentCleanup(OrderNo: Code[20]) // BAD EXAMPLE
    var
        SalesHeader: Record "Sales Header";
        SalesLine: Record "Sales Line";
        SalesCommentLine: Record "Sales Comment Line";
    begin
        // PROBLEM: Mixing deletion methods without understanding implications

        // Uses DeleteAll (bypasses events)
        SalesLine.SetRange("Document No.", OrderNo);
        SalesLine.DeleteAll(); // OnDelete events NOT called

        // Uses iterative deletion (processes events)
        SalesCommentLine.SetRange("Document Type", SalesCommentLine."Document Type"::Order);
        SalesCommentLine.SetRange("No.", OrderNo);
        if SalesCommentLine.FindSet() then
            repeat
                SalesCommentLine.Delete(); // OnDelete events ARE called
            until SalesCommentLine.Next() = 0;

        // Manual deletion (bypasses AL Delete method entirely)
        SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo);
        SalesHeader.Delete(); // OnDelete events ARE called

        // PROBLEMS:
        // 1. Inconsistent event processing across related tables
        // 2. Potential data integrity issues from partial event execution
        // 3. Unpredictable behavior for consuming code
    end;

    procedure UnoptimizedBulkDeletion() // BAD EXAMPLE
    var
        Item: Record Item;
    begin
        // ANTI-PATTERN: Using loop when DeleteAll would be more efficient
        Item.SetRange(Blocked, true);
        Item.SetFilter("Last Date Modified", '<%1', CalcDate('-1Y', Today));

        // INEFFICIENT: Creates individual SQL DELETE statements
        if Item.FindSet() then
            repeat
                Item.Delete(); // Each iteration = separate SQL statement
            until Item.Next() = 0;

        // BETTER APPROACH: Single SQL operation
        // Item.DeleteAll(); // One SQL DELETE statement
    end;
}

/*
CORRECT DELETEALL USAGE PRINCIPLES:

1. Consistency: Use same deletion method for related operations
2. Event Awareness: Understand which events are bypassed
3. Performance Priority: Choose DeleteAll for bulk operations
4. Documentation: Clearly document event bypass behavior
5. Validation: Ensure database constraints handle integrity
6. Testing: Verify behavior matches expectations in all scenarios

DECISION MATRIX:
- Large datasets + No critical AL events = DeleteAll
- Small datasets + Complex AL logic = Iterative deletion
- Mixed requirements = Separate phases or manual event processing
*/
```