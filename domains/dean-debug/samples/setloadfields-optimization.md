# SetLoadFields Performance Optimization - AL Code Examples

## Basic SetLoadFields Implementation

```al
// Optimized: Load only required fields
procedure GetCustomerNameOptimized(CustomerNo: Code[20]): Text[100]
var
    Customer: Record Customer;
begin
    Customer.SetLoadFields(Name);
    if Customer.Get(CustomerNo) then
        exit(Customer.Name);
    exit('');
end;

// Non-optimized: Loads all fields
procedure GetCustomerNameUnoptimized(CustomerNo: Code[20]): Text[100]
var
    Customer: Record Customer;
begin
    if Customer.Get(CustomerNo) then
        exit(Customer.Name);
    exit('');
end;
```

## Bulk Operations Optimization

```al
procedure ProcessCustomerListOptimized()
var
    Customer: Record Customer;
    ProcessedCount: Integer;
begin
    Customer.SetLoadFields("No.", Name, "Customer Posting Group", Blocked);

    if Customer.FindSet() then
        repeat
            if not Customer.Blocked then begin
                ProcessCustomerRecord(Customer."No.", Customer.Name, Customer."Customer Posting Group");
                ProcessedCount += 1;
            end;
        until Customer.Next() = 0;

    Message('Processed %1 customers with optimized field loading', ProcessedCount);
end;

procedure ProcessCustomerListUnoptimized()
var
    Customer: Record Customer;
    ProcessedCount: Integer;
begin
    if Customer.FindSet() then
        repeat
            if not Customer.Blocked then begin
                ProcessCustomerRecord(Customer."No.", Customer.Name, Customer."Customer Posting Group");
                ProcessedCount += 1;
            end;
        until Customer.Next() = 0;

    Message('Processed %1 customers with full field loading', ProcessedCount);
end;
```

## Report Generation with Selective Loading

```al
procedure GenerateCustomerReportOptimized()
var
    Customer: Record Customer;
    ReportData: JsonObject;
    CustomerArray: JsonArray;
    CustomerJson: JsonObject;
begin
    Customer.SetLoadFields("No.", Name, "Phone No.", "E-Mail", "Country/Region Code", "Credit Limit (LCY)");

    if Customer.FindSet() then
        repeat
            Clear(CustomerJson);
            CustomerJson.Add('number', Customer."No.");
            CustomerJson.Add('name', Customer.Name);
            CustomerJson.Add('phone', Customer."Phone No.");
            CustomerJson.Add('email', Customer."E-Mail");
            CustomerJson.Add('country', Customer."Country/Region Code");
            CustomerJson.Add('creditLimit', Customer."Credit Limit (LCY)");
            CustomerArray.Add(CustomerJson);
        until Customer.Next() = 0;

    ReportData.Add('customers', CustomerArray);
end;
```

## Advanced Memory Optimization

```al
procedure ProcessSalesOrdersWithItems()
var
    SalesHeader: Record "Sales Header";
    SalesLine: Record "Sales Line";
    OrderCount: Integer;
    LineCount: Integer;
begin
    SalesHeader.SetLoadFields("No.", "Sell-to Customer No.", "Order Date", "Amount Including VAT");
    SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);

    if SalesHeader.FindSet() then
        repeat
            OrderCount += 1;

            SalesLine.SetLoadFields("Document No.", "Line No.", "No.", Quantity, "Unit Price", "Line Amount");
            SalesLine.SetRange("Document Type", SalesLine."Document Type"::Order);
            SalesLine.SetRange("Document No.", SalesHeader."No.");

            if SalesLine.FindSet() then
                repeat
                    LineCount += 1;
                    ProcessSalesLineData(SalesLine."No.", SalesLine.Quantity, SalesLine."Unit Price");
                until SalesLine.Next() = 0;

        until SalesHeader.Next() = 0;

    Message('Processed %1 orders with %2 lines using optimized field loading', OrderCount, LineCount);
end;
```

## Conditional Field Loading

```al
procedure ProcessCustomersConditional(IncludeFinancialData: Boolean; IncludeContactData: Boolean)
var
    Customer: Record Customer;
    FieldList: List of [Integer];
begin
    FieldList.Add(Customer.FieldNo("No."));
    FieldList.Add(Customer.FieldNo(Name));

    if IncludeFinancialData then begin
        FieldList.Add(Customer.FieldNo("Credit Limit (LCY)"));
        FieldList.Add(Customer.FieldNo("Balance (LCY)"));
        FieldList.Add(Customer.FieldNo("Sales (LCY)"));
    end;

    if IncludeContactData then begin
        FieldList.Add(Customer.FieldNo("Phone No."));
        FieldList.Add(Customer.FieldNo("E-Mail"));
        FieldList.Add(Customer.FieldNo("Contact"));
    end;

    Customer.SetLoadFields(FieldList);

    if Customer.FindSet() then
        repeat
            ProcessCustomerConditionally(Customer, IncludeFinancialData, IncludeContactData);
        until Customer.Next() = 0;
end;
```

## Performance Comparison

```al
procedure CompareMemoryUsage()
var
    Customer: Record Customer;
    StartTime: DateTime;
    EndTime: DateTime;
    RecordCount: Integer;
begin
    // Test 1: With SetLoadFields optimization
    StartTime := CurrentDateTime;
    Customer.SetLoadFields("No.", Name);
    if Customer.FindSet() then
        repeat
            RecordCount += 1;
            ProcessBasicCustomerInfo(Customer."No.", Customer.Name);
        until Customer.Next() = 0;
    EndTime := CurrentDateTime;

    Message('Optimized: Processed %1 records in %2 ms', RecordCount, EndTime - StartTime);

    // Test 2: Without SetLoadFields
    Clear(Customer);
    RecordCount := 0;
    StartTime := CurrentDateTime;
    if Customer.FindSet() then
        repeat
            RecordCount += 1;
            ProcessBasicCustomerInfo(Customer."No.", Customer.Name);
        until Customer.Next() = 0;
    EndTime := CurrentDateTime;

    Message('Unoptimized: Processed %1 records in %2 ms', RecordCount, EndTime - StartTime);
end;
```

## Query Efficiency Demonstration

```al
procedure DemonstrateQueryEfficiency()
var
    Item: Record Item;
    ItemCount: Integer;
    FilteredCount: Integer;
begin
    Item.SetLoadFields("No.", Description, "Unit Price", "Inventory Posting Group", Blocked);
    Item.SetFilter("Unit Price", '>%1', 100);
    Item.SetRange(Blocked, false);

    if Item.FindSet() then
        repeat
            ItemCount += 1;
            if StrPos(Item.Description, 'PREMIUM') > 0 then begin
                FilteredCount += 1;
                ProcessPremiumItem(Item."No.", Item.Description, Item."Unit Price");
            end;
        until Item.Next() = 0;

    Message('Found %1 premium items from %2 total items using optimized loading', FilteredCount, ItemCount);
end;
```

## Anti-Pattern Examples

```al
// ❌ ANTI-PATTERN: Loading unnecessary fields
procedure AntiPatternUnnecessaryFields()
var
    Customer: Record Customer;
begin
    Customer.SetLoadFields("No.", Name, "Phone No.", "E-Mail", "Address", "City",
                          "Credit Limit (LCY)", "Balance (LCY)", "Sales (LCY)");

    if Customer.FindSet() then
        repeat
            ProcessBasicCustomerInfo(Customer."No.", Customer.Name);
        until Customer.Next() = 0;
end;

// ❌ ANTI-PATTERN: Not using SetLoadFields in loops
procedure AntiPatternNoOptimizationInLoops()
var
    SalesHeader: Record "Sales Header";
    Customer: Record Customer;
begin
    if SalesHeader.FindSet() then
        repeat
            if Customer.Get(SalesHeader."Sell-to Customer No.") then
                ProcessCustomerInOrder(Customer.Name, Customer."Credit Limit (LCY)");
        until SalesHeader.Next() = 0;
end;

// ✅ CORRECTED: Proper SetLoadFields usage
procedure CorrectedOptimizationInLoops()
var
    SalesHeader: Record "Sales Header";
    Customer: Record Customer;
begin
    SalesHeader.SetLoadFields("No.", "Sell-to Customer No.");

    if SalesHeader.FindSet() then
        repeat
            Customer.SetLoadFields(Name, "Credit Limit (LCY)");
            if Customer.Get(SalesHeader."Sell-to Customer No.") then
                ProcessCustomerInOrder(Customer.Name, Customer."Credit Limit (LCY)");
        until SalesHeader.Next() = 0;
end;
```

## Performance Impact Measurement

```al
procedure MeasureAntiPatternImpact()
var
    Item: Record Item;
    ItemLedgerEntry: Record "Item Ledger Entry";
    StartTime: DateTime;
    EndTime: DateTime;
    EntryCount: Integer;
begin
    StartTime := CurrentDateTime;

    if ItemLedgerEntry.FindSet() then
        repeat
            EntryCount += 1;
            if Item.Get(ItemLedgerEntry."Item No.") then begin
                ProcessItemLedgerWithDescription(ItemLedgerEntry."Entry No.", Item.Description);
            end;
        until (ItemLedgerEntry.Next() = 0) or (EntryCount > 1000);

    EndTime := CurrentDateTime;
    Message('Anti-pattern: Processed %1 entries in %2 ms', EntryCount, EndTime - StartTime);

    ProcessItemLedgerOptimized();
end;

local procedure ProcessItemLedgerOptimized()
var
    Item: Record Item;
    ItemLedgerEntry: Record "Item Ledger Entry";
    StartTime: DateTime;
    EndTime: DateTime;
    EntryCount: Integer;
begin
    StartTime := CurrentDateTime;

    ItemLedgerEntry.SetLoadFields("Entry No.", "Item No.");
    Item.SetLoadFields(Description);

    if ItemLedgerEntry.FindSet() then
        repeat
            EntryCount += 1;
            if Item.Get(ItemLedgerEntry."Item No.") then
                ProcessItemLedgerWithDescription(ItemLedgerEntry."Entry No.", Item.Description);
        until (ItemLedgerEntry.Next() = 0) or (EntryCount > 1000);

    EndTime := CurrentDateTime;
    Message('Optimized: Processed %1 entries in %2 ms', EntryCount, EndTime - StartTime);
end;
```

## Best Practice Implementation

```al
procedure BestPracticeImplementation()
var
    Customer: Record Customer;
    ProcessedCount: Integer;
    ErrorCount: Integer;
begin
    Customer.SetLoadFields("No.", Name, "Customer Posting Group", Blocked);

    if Customer.FindSet() then
        repeat
            if ProcessCustomerSafely(Customer) then
                ProcessedCount += 1
            else
                ErrorCount += 1;
        until Customer.Next() = 0;

    if ErrorCount > 0 then
        Message('Processed %1 customers successfully, %2 errors encountered', ProcessedCount, ErrorCount);
end;

local procedure ProcessCustomerSafely(Customer: Record Customer): Boolean
begin
    if (Customer."No." = '') or (Customer.Name = '') then
        exit(false);

    if Customer.Blocked then
        exit(true);

    ProcessBasicCustomerInfo(Customer."No.", Customer.Name);
    exit(true);
end;
```

## Dynamic Field Loading

```al
procedure DynamicFieldLoading(ProcessingMode: Option Basic,Extended,Full)
var
    Customer: Record Customer;
    FieldsToLoad: List of [Integer];
begin
    FieldsToLoad.Add(Customer.FieldNo("No."));
    FieldsToLoad.Add(Customer.FieldNo(Name));

    case ProcessingMode of
        ProcessingMode::Extended:
            begin
                FieldsToLoad.Add(Customer.FieldNo("Phone No."));
                FieldsToLoad.Add(Customer.FieldNo("E-Mail"));
            end;
        ProcessingMode::Full:
            begin
                FieldsToLoad.Add(Customer.FieldNo("Phone No."));
                FieldsToLoad.Add(Customer.FieldNo("E-Mail"));
                FieldsToLoad.Add(Customer.FieldNo("Credit Limit (LCY)"));
                FieldsToLoad.Add(Customer.FieldNo("Balance (LCY)"));
            end;
    end;

    Customer.SetLoadFields(FieldsToLoad);

    if Customer.FindSet() then
        repeat
            ProcessCustomerByMode(Customer, ProcessingMode);
        until Customer.Next() = 0;
end;
```

## Performance Guidelines Summary

### Implementation Strategy
1. **Analyze field usage** before applying SetLoadFields
2. **Load minimal field sets** required for processing
3. **Apply early** - call SetLoadFields immediately after record declaration
4. **Measure impact** - validate performance improvements
5. **Document rationale** - explain field selection decisions

### Memory Optimization Benefits
- 70-90% memory reduction for large recordsets
- Improved garbage collection performance
- Better system scalability with reduced memory pressure
- Faster record processing with smaller data structures

### Common Mistakes to Avoid
1. Loading fields "just in case" without actual usage
2. Forgetting SetLoadFields in nested record lookups
3. Excluding primary key fields needed for record operations
4. Applying optimization to small datasets where overhead exceeds benefits
5. Not measuring actual performance improvement