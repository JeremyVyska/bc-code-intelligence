# Lonely Repeat Statement Pattern - AL Code Examples

## Overview
The "Lonely Repeat Statement Pattern" refers to repeat-until loops that are difficult to read due to poor formatting, unclear conditions, or lack of proper structure. This sample demonstrates how to identify and fix these patterns in AL code.

## Problem Pattern: Lonely and Hard-to-Read Repeat Statements

### Example 1: Basic Formatting Issues

**Problematic Code:**
```al
procedure ProcessCustomerRecords()
var
    Customer: Record Customer;
    Counter: Integer;
begin
    Customer.FindFirst();
    repeat Customer.CalcFields("Sales (LCY)"); if Customer."Sales (LCY)" > 10000 then begin Counter += 1; Message('High value customer: %1', Customer.Name); end; until Customer.Next() = 0;
end;
```

**Improved Code:**
```al
procedure ProcessCustomerRecords()
var
    Customer: Record Customer;
    Counter: Integer;
begin
    if Customer.FindFirst() then
        repeat
            Customer.CalcFields("Sales (LCY)");
            if Customer."Sales (LCY)" > 10000 then begin
                Counter += 1;
                Message('High value customer: %1', Customer.Name);
            end;
        until Customer.Next() = 0;
end;
```

### Example 2: Complex Condition Handling

**Problematic Code:**
```al
procedure ValidateItemInventory()
var
    Item: Record Item;
    InventorySetup: Record "Inventory Setup";
    IsValid: Boolean;
begin
    Item.SetRange(Blocked, false);
    Item.FindFirst();
    repeat IsValid := true; Item.CalcFields(Inventory); if (Item.Inventory < Item."Reorder Point") and (Item."Reorder Point" > 0) and (Item."Replenishment System" = Item."Replenishment System"::Purchase) then IsValid := false; if not IsValid then Error('Item %1 needs reordering', Item."No."); until Item.Next() = 0;
end;
```

**Improved Code:**
```al
procedure ValidateItemInventory()
var
    Item: Record Item;
    NeedsReordering: Boolean;
begin
    Item.SetRange(Blocked, false);
    if Item.FindFirst() then
        repeat
            Item.CalcFields(Inventory);

            NeedsReordering := (Item.Inventory < Item."Reorder Point") and
                              (Item."Reorder Point" > 0) and
                              (Item."Replenishment System" = Item."Replenishment System"::Purchase);

            if NeedsReordering then
                Error('Item %1 needs reordering', Item."No.");
        until Item.Next() = 0;
end;
```

## Advanced Pattern Improvements

### Example 3: Nested Loop Clarity

**Problematic Code:**
```al
procedure RecalculateCustomerStatistics()
var
    Customer: Record Customer;
    SalesLine: Record "Sales Line";
    TotalAmount: Decimal;
begin
    Customer.FindFirst();
    repeat TotalAmount := 0; SalesLine.SetRange("Sell-to Customer No.", Customer."No."); if SalesLine.FindFirst() then repeat TotalAmount += SalesLine."Line Amount"; until SalesLine.Next() = 0; Customer."Sales (LCY)" := TotalAmount; Customer.Modify(); until Customer.Next() = 0;
end;
```

**Improved Code:**
```al
procedure RecalculateCustomerStatistics()
var
    Customer: Record Customer;
    SalesLine: Record "Sales Line";
    TotalAmount: Decimal;
begin
    if Customer.FindFirst() then
        repeat
            TotalAmount := 0;
            SalesLine.SetRange("Sell-to Customer No.", Customer."No.");

            if SalesLine.FindFirst() then
                repeat
                    TotalAmount += SalesLine."Line Amount";
                until SalesLine.Next() = 0;

            Customer."Sales (LCY)" := TotalAmount;
            Customer.Modify();
        until Customer.Next() = 0;
end;
```

### Example 4: Error Handling in Repeat Loops

**Problematic Code:**
```al
procedure PostSalesDocuments()
var
    SalesHeader: Record "Sales Header";
    SalesPost: Codeunit "Sales-Post";
begin
    SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
    SalesHeader.SetRange(Status, SalesHeader.Status::Released);
    SalesHeader.FindFirst();
    repeat if not SalesPost.Run(SalesHeader) then Message('Error posting document %1', SalesHeader."No."); until SalesHeader.Next() = 0;
end;
```

**Improved Code:**
```al
procedure PostSalesDocuments()
var
    SalesHeader: Record "Sales Header";
    SalesPost: Codeunit "Sales-Post";
    DocumentsProcessed: Integer;
    DocumentsFailed: Integer;
begin
    SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
    SalesHeader.SetRange(Status, SalesHeader.Status::Released);

    if SalesHeader.FindFirst() then
        repeat
            Clear(SalesPost);
            if SalesPost.Run(SalesHeader) then
                DocumentsProcessed += 1
            else begin
                DocumentsFailed += 1;
                Message('Error posting document %1: %2', SalesHeader."No.", GetLastErrorText());
            end;
        until SalesHeader.Next() = 0;

    Message('Processing complete. Posted: %1, Failed: %2', DocumentsProcessed, DocumentsFailed);
end;
```

## Performance and Safety Patterns

### Example 5: Large Dataset Processing

**Problematic Code:**
```al
procedure ArchiveOldEntries()
var
    GLEntry: Record "G/L Entry";
    ArchiveEntry: Record "G/L Entry Archive";
begin
    GLEntry.SetFilter("Posting Date", '<%1', CalcDate('-2Y', Today()));
    GLEntry.FindFirst();
    repeat ArchiveEntry.TransferFields(GLEntry); ArchiveEntry.Insert(); GLEntry.Delete(); until GLEntry.Next() = 0;
end;
```

**Improved Code:**
```al
procedure ArchiveOldEntries()
var
    GLEntry: Record "G/L Entry";
    ArchiveEntry: Record "G/L Entry Archive";
    ProcessedCount: Integer;
    BatchSize: Integer;
begin
    BatchSize := 1000; // Process in batches for better performance
    ProcessedCount := 0;

    GLEntry.SetFilter("Posting Date", '<%1', CalcDate('-2Y', Today()));

    if GLEntry.FindFirst() then
        repeat
            ArchiveEntry.TransferFields(GLEntry);
            ArchiveEntry.Insert();
            GLEntry.Delete();

            ProcessedCount += 1;

            // Commit periodically to avoid long transactions
            if ProcessedCount mod BatchSize = 0 then begin
                Commit();
                Message('Processed %1 entries...', ProcessedCount);
            end;
        until GLEntry.Next() = 0;

    Message('Archive complete. Total entries processed: %1', ProcessedCount);
end;
```

### Example 6: Conditional Processing with Early Exit

**Problematic Code:**
```al
procedure FindFirstAvailableItem(LocationCode: Code[10]): Code[20]
var
    Item: Record Item;
    ItemLedgerEntry: Record "Item Ledger Entry";
    AvailableQty: Decimal;
begin
    Item.FindFirst();
    repeat Item.SetRange("Location Filter", LocationCode); Item.CalcFields(Inventory); if Item.Inventory > 0 then exit(Item."No."); until Item.Next() = 0;
end;
```

**Improved Code:**
```al
procedure FindFirstAvailableItem(LocationCode: Code[10]): Code[20]
var
    Item: Record Item;
begin
    Item.SetRange(Blocked, false);
    Item.SetRange(Type, Item.Type::Inventory);

    if Item.FindFirst() then
        repeat
            Item.SetRange("Location Filter", LocationCode);
            Item.CalcFields(Inventory);

            if Item.Inventory > 0 then
                exit(Item."No.");
        until Item.Next() = 0;

    // Return empty if no available item found
    exit('');
end;
```

## Best Practices Summary

### Formatting Guidelines
1. **Always use proper indentation** for repeat-until blocks
2. **Place conditions on separate lines** for complex boolean expressions
3. **Use meaningful variable names** instead of inline calculations
4. **Add blank lines** to separate logical sections within loops

### Performance Considerations
1. **Process in batches** for large datasets to avoid transaction timeouts
2. **Use proper filters** before starting repeat loops to minimize iterations
3. **Calculate fields only when necessary** within the loop
4. **Consider using FindSet()** with ReadOnly parameter for read-only operations

### Error Handling
1. **Always check FindFirst() result** before starting repeat loop
2. **Handle errors gracefully** with appropriate user feedback
3. **Use Clear()** on codeunit variables before calling Run()
4. **Implement progress indicators** for long-running operations

### Code Readability
1. **Extract complex conditions** into well-named boolean variables
2. **Use early exit patterns** when searching for specific records
3. **Add comments** to explain business logic within loops
4. **Avoid deeply nested structures** by using helper procedures

## Related Patterns
- **FindSet Pattern**: Use `FindSet()` with `ReadOnly` parameter for better performance when only reading data
- **Batch Processing Pattern**: Process records in controlled batches with periodic commits
- **Error Accumulation Pattern**: Collect errors during processing and report them together
- **Progress Indication Pattern**: Provide user feedback during long-running repeat operations