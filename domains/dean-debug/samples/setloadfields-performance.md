# SetLoadFields Performance Optimization - AL Examples

## Basic Usage Pattern

```al
codeunit 50100 "SetLoadFields Examples"
{
    // Basic SetLoadFields usage for loading only needed fields
    procedure GetCustomerNameAndPhone(CustomerNo: Code[20]): Text
    var
        Customer: Record Customer;
        Result: Text;
    begin
        Customer.Get(CustomerNo);

        // Load only the fields we actually need
        Customer.SetLoadFields(Name, "Phone No.");
        Customer.Find(); // Refresh with partial loading

        Result := Customer.Name + ' - ' + Customer."Phone No.";
        exit(Result);
    end;
}
```

## Performance Impact Demonstration

### Before: Full Record Loading
```al
// Traditional approach - loads all 100+ fields from Customer table
procedure ProcessCustomerListSlow()
var
    Customer: Record Customer;
    ProcessedCount: Integer;
begin
    // This loads ALL fields for each customer record
    // Network overhead: ~8KB per record for full Customer table
    // Memory usage: Significant for large datasets
    if Customer.FindSet() then
        repeat
            // Only using Name and "No." but loaded entire record
            if Customer.Name <> '' then
                ProcessedCount += 1;
        until Customer.Next() = 0;

    Message('Processed %1 customers (slow method)', ProcessedCount);
end;
```

### After: Optimized with SetLoadFields
```al
// Optimized approach - loads only required fields
procedure ProcessCustomerListFast()
var
    Customer: Record Customer;
    ProcessedCount: Integer;
begin
    // Load only the fields we actually use
    Customer.SetLoadFields("No.", Name);

    // Network overhead: ~50 bytes per record (vs 8KB)
    // Memory usage: 99% reduction for this scenario
    // Performance gain: 10-50x faster depending on table size
    if Customer.FindSet() then
        repeat
            if Customer.Name <> '' then
                ProcessedCount += 1;
        until Customer.Next() = 0;

    Message('Processed %1 customers (fast method)', ProcessedCount);
end;
```

## Anti-Pattern: Field Access After SetLoadFields

```al
// WRONG: Accessing fields not included in SetLoadFields
procedure CustomerAnalysisAntiPattern(CustomerNo: Code[20])
var
    Customer: Record Customer;
begin
    Customer.Get(CustomerNo);

    // Only load basic fields
    Customer.SetLoadFields("No.", Name);
    Customer.Find();

    // BAD: Accessing field not in SetLoadFields causes additional database hit
    // This negates the performance benefit and adds overhead
    if Customer."Credit Limit (LCY)" > 0 then  // Database hit here!
        Message('Customer %1 has credit limit', Customer.Name);

    // WORSE: Multiple unloaded field accesses = multiple database hits
    Customer."Phone No.";           // Database hit!
    Customer."E-Mail";              // Another database hit!
    Customer."VAT Registration No."; // Yet another database hit!
end;

// CORRECT: Include all needed fields in SetLoadFields
procedure CustomerAnalysisCorrectPattern(CustomerNo: Code[20])
var
    Customer: Record Customer;
begin
    Customer.Get(CustomerNo);

    // Load ALL fields we plan to use
    Customer.SetLoadFields("No.", Name, "Credit Limit (LCY)", "Phone No.", "E-Mail", "VAT Registration No.");
    Customer.Find();

    // Now all field accesses are served from memory - no additional database hits
    if Customer."Credit Limit (LCY)" > 0 then
        Message('Customer %1 (%2) has credit limit. Contact: %3',
                Customer.Name, Customer."VAT Registration No.", Customer."E-Mail");
end;
```

## Batch Processing Optimization

```al
// SetLoadFields optimization for processing large datasets
procedure OptimizeBatchCustomerUpdate()
var
    Customer: Record Customer;
    SalesHeader: Record "Sales Header";
    UpdateCount: Integer;
begin
    // Scenario: Update customer last sale date based on sales orders
    // Performance critical when processing thousands of records

    // Step 1: Optimize customer record loading
    Customer.SetLoadFields("No.", "Last Date Modified");

    if Customer.FindSet(true) then  // Modifiable RecordSet
        repeat
            // Step 2: Optimize sales header lookup - only need posting date
            SalesHeader.SetRange("Sell-to Customer No.", Customer."No.");
            SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
            SalesHeader.SetLoadFields("Posting Date");

            if SalesHeader.FindLast() then begin
                // Only modify if there's an actual change
                if Customer."Last Date Modified" <> SalesHeader."Posting Date" then begin
                    Customer."Last Date Modified" := SalesHeader."Posting Date";
                    Customer.Modify();  // Only updates loaded fields + modified fields
                    UpdateCount += 1;
                end;
            end;
        until Customer.Next() = 0;

    Message('Updated %1 customer records', UpdateCount);
end;
```

## Performance Monitoring Example

```al
// Measure SetLoadFields impact
procedure MeasureSetLoadFieldsImpact()
var
    Customer: Record Customer;
    StartTime: DateTime;
    EndTime: DateTime;
    RecordCount: Integer;
begin
    // Test 1: Traditional full record loading
    StartTime := CurrentDateTime;
    RecordCount := 0;

    if Customer.FindSet() then
        repeat
            RecordCount += 1;
        until Customer.Next() = 0;

    EndTime := CurrentDateTime;
    Message('Full loading: %1 records in %2ms', RecordCount, EndTime - StartTime);

    // Test 2: Optimized with SetLoadFields
    StartTime := CurrentDateTime;
    RecordCount := 0;
    Customer.Reset();
    Customer.SetLoadFields("No.");  // Minimal field loading

    if Customer.FindSet() then
        repeat
            RecordCount += 1;
        until Customer.Next() = 0;

    EndTime := CurrentDateTime;
    Message('SetLoadFields: %1 records in %2ms', RecordCount, EndTime - StartTime);

    // Typical results: 70-90% performance improvement for large datasets
end;
```

**Performance Guidelines:**

1. **SetLoadFields Benefits**:
   - Network traffic reduction: 80-95% for wide tables
   - Memory usage reduction: Proportional to excluded fields
   - Query performance: Faster due to less data transfer

2. **Best Practices**:
   - Always include primary key fields
   - Plan field usage before calling SetLoadFields
   - Use SetLoadFields before FindSet() for maximum benefit
   - Avoid accessing non-loaded fields after SetLoadFields

3. **Performance Impact**:
   - Small datasets (<1000 records): 20-50% improvement
   - Medium datasets (1000-10000 records): 50-300% improvement
   - Large datasets (>10000 records): 300-1000% improvement