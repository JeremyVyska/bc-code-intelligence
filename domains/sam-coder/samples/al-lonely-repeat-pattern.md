# AL Lonely Repeat Pattern - Code Examples

## Good Examples

### Proper Repeat-Until Loop Structure
```al
procedure ProcessCustomerRecords()
var
    Customer: Record Customer;
    ProcessedCount: Integer;
begin
    ProcessedCount := 0;

    if Customer.FindSet() then
        repeat
            // Process each customer record
            if not Customer.Blocked then begin
                Customer."Last Contact Date" := Today;
                Customer.Modify();
                ProcessedCount += 1;
            end;
        until Customer.Next() = 0;  // Proper until condition

    Message('Processed %1 customer records', ProcessedCount);
end;
```

### Repeat Loop with Error Handling and Exit Conditions
```al
procedure SafeDataProcessing()
var
    DataRecord: Record "Data Processing Queue";
    MaxAttempts: Integer;
    AttemptCount: Integer;
    ProcessingComplete: Boolean;
begin
    MaxAttempts := 5;
    AttemptCount := 0;
    ProcessingComplete := false;

    if DataRecord.FindSet() then
        repeat
            AttemptCount += 1;

            // Process with error handling
            if TryProcessRecord(DataRecord) then begin
                ProcessingComplete := true;
                DataRecord."Processing Status" := DataRecord."Processing Status"::Completed;
                DataRecord.Modify();
            end else begin
                DataRecord."Error Count" += 1;
                DataRecord.Modify();
            end;

        until (DataRecord.Next() = 0) or (AttemptCount >= MaxAttempts) or ProcessingComplete;

    if not ProcessingComplete then
        Error('Data processing failed after %1 attempts', MaxAttempts);
end;
```

### Complex Repeat Loop with Multiple Exit Conditions
```al
procedure BatchOrderProcessing()
var
    SalesHeader: Record "Sales Header";
    ProcessingStartTime: Time;
    MaxProcessingTime: Duration;
    OrdersProcessed: Integer;
    BatchSize: Integer;
begin
    ProcessingStartTime := Time;
    MaxProcessingTime := 300000;  // 5 minutes in milliseconds
    OrdersProcessed := 0;
    BatchSize := 100;

    SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
    SalesHeader.SetRange(Status, SalesHeader.Status::Open);

    if SalesHeader.FindSet() then
        repeat
            // Process individual order
            if ProcessSingleOrder(SalesHeader) then
                OrdersProcessed += 1;

            // Update progress every 10 orders
            if OrdersProcessed mod 10 = 0 then
                UpdateProgressIndicator(OrdersProcessed);

        until (SalesHeader.Next() = 0) or
              (OrdersProcessed >= BatchSize) or
              (Time - ProcessingStartTime > MaxProcessingTime);

    Message('Batch processing complete. Processed %1 orders', OrdersProcessed);
end;
```

### Repeat Loop with Data Validation
```al
procedure ValidateAndProcessItems()
var
    Item: Record Item;
    ValidationErrors: Integer;
    TotalItems: Integer;
begin
    ValidationErrors := 0;
    TotalItems := 0;

    Item.SetRange(Blocked, false);

    if Item.FindSet() then
        repeat
            TotalItems += 1;

            // Validate each item
            if not ValidateItemData(Item) then begin
                ValidationErrors += 1;
                LogValidationError(Item."No.");
            end else begin
                Item."Last Validation Date" := Today;
                Item.Modify();
            end;

        until Item.Next() = 0;

    if ValidationErrors > 0 then
        Message('Validation complete: %1 errors found out of %2 items', ValidationErrors, TotalItems)
    else
        Message('All %1 items validated successfully', TotalItems);
end;
```

## Bad Examples

### Lonely Repeat Without Until
```al
procedure LonelyRepeat()
var
    Customer: Record Customer;
begin
    if Customer.FindSet() then
        repeat
            Customer."Last Contact Date" := Today;
            Customer.Modify();
        // Missing until condition - this creates an infinite loop!

    Message('Processing complete');  // This will never execute
end;
```

### Unreachable Until Condition
```al
procedure UnreachableUntil()
var
    Counter: Integer;
    MaxCount: Integer;
begin
    Counter := 1;
    MaxCount := 10;

    repeat
        Message('Processing item %1', Counter);
        // Counter is never incremented - until condition can never be met
    until Counter > MaxCount;  // This creates an infinite loop
end;
```

### Complex Until Logic That Obscures Termination
```al
procedure ObscureTermination()
var
    SalesLine: Record "Sales Line";
    ComplexCondition: Boolean;
    AnotherCondition: Boolean;
    YetAnotherCondition: Boolean;
begin
    if SalesLine.FindSet() then
        repeat
            // Complex processing logic
            ComplexCondition := (SalesLine.Quantity > 0) and (SalesLine."Unit Price" > 0);
            AnotherCondition := (SalesLine."Line Discount %" < 50) or (SalesLine.Type = SalesLine.Type::Item);
            YetAnotherCondition := not ((SalesLine."Shipment Date" = 0D) and (SalesLine."Qty. to Ship" > 0));

            ProcessSalesLine(SalesLine);

        until (SalesLine.Next() = 0) and ComplexCondition and AnotherCondition and YetAnotherCondition;  // Poor: complex until condition
end;
```

### No Loop Variable Progression
```al
procedure NoProgression()
var
    ItemNo: Code[20];
    Item: Record Item;
    FoundItem: Boolean;
begin
    ItemNo := 'ITEM001';
    FoundItem := false;

    repeat
        if Item.Get(ItemNo) then begin
            FoundItem := true;
            ProcessItem(Item);
        end;
        // ItemNo never changes - if item doesn't exist, this becomes infinite loop
    until FoundItem;
end;
```

### Missing Error Handling for Loop Failures
```al
procedure NoErrorHandling()
var
    DataRecord: Record "Processing Queue";
    ProcessingSuccess: Boolean;
begin
    ProcessingSuccess := false;

    if DataRecord.FindSet() then
        repeat
            // Processing that might fail
            ProcessRecord(DataRecord);  // No error handling
            ProcessingSuccess := true;
        until DataRecord.Next() = 0;

    // No handling for case where processing fails
    Message('Processing complete');
end;
```

## Best Practices

1. **Always pair repeat with until** - never leave repeat statements incomplete
2. **Ensure until conditions can be met** - validate that loop variables progress toward termination
3. **Include multiple exit strategies** when appropriate (time limits, error counts, batch sizes)
4. **Use clear, simple until conditions** that are easy to understand and verify
5. **Add error handling** to prevent infinite loops from processing failures
6. **Validate loop progression** to ensure forward movement toward termination condition