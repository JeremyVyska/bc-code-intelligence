# Record Find Early Exit - AL Code Examples

## Good Examples

### Early Exit with Find Pattern
```al
procedure GetCustomerName(CustomerNo: Code[20]): Text[100]
var
    Customer: Record Customer;
begin
    // Early exit - avoid unnecessary processing
    if not Customer.Get(CustomerNo) then
        exit('');

    // Only execute if record found
    exit(Customer.Name);
end;
```

### Complex Find with Early Validation
```al
procedure ProcessActiveCustomerOrders(CustomerNo: Code[20]): Boolean
var
    Customer: Record Customer;
    SalesHeader: Record "Sales Header";
    ProcessedCount: Integer;
begin
    // Early exit for invalid customer
    if CustomerNo = '' then
        exit(false);

    // Early exit if customer doesn't exist
    if not Customer.Get(CustomerNo) then
        exit(false);

    // Early exit if customer is blocked
    if Customer.Blocked <> Customer.Blocked::" " then
        exit(false);

    // Process orders only after all validations pass
    SalesHeader.SetRange("Sell-to Customer No.", CustomerNo);
    SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
    SalesHeader.SetRange(Status, SalesHeader.Status::Open);

    if not SalesHeader.FindSet() then
        exit(false);

    repeat
        if ProcessSalesOrder(SalesHeader."No.") then
            ProcessedCount += 1;
    until SalesHeader.Next() = 0;

    exit(ProcessedCount > 0);
end;
```

### FindFirst with Early Exit
```al
procedure GetLatestInvoiceAmount(CustomerNo: Code[20]): Decimal
var
    SalesInvoiceHeader: Record "Sales Invoice Header";
begin
    SalesInvoiceHeader.SetRange("Sell-to Customer No.", CustomerNo);
    SalesInvoiceHeader.SetCurrentKey("Posting Date");
    SalesInvoiceHeader.Ascending(false);

    // Early exit if no invoices found
    if not SalesInvoiceHeader.FindFirst() then
        exit(0);

    SalesInvoiceHeader.CalcFields("Amount Including VAT");
    exit(SalesInvoiceHeader."Amount Including VAT");
end;
```

### Multiple Table Lookup Optimization
```al
procedure ValidateItemAvailability(ItemNo: Code[20]; LocationCode: Code[10]; RequiredQty: Decimal): Boolean
var
    Item: Record Item;
    Location: Record Location;
    ItemLedgerEntry: Record "Item Ledger Entry";
    AvailableQty: Decimal;
begin
    // Early validation - item must exist
    if not Item.Get(ItemNo) then
        exit(false);

    // Early exit if item is blocked
    if Item.Blocked then
        exit(false);

    // Early validation - location must exist if specified
    if (LocationCode <> '') and (not Location.Get(LocationCode)) then
        exit(false);

    // Calculate available quantity only after validations
    ItemLedgerEntry.SetRange("Item No.", ItemNo);
    if LocationCode <> '' then
        ItemLedgerEntry.SetRange("Location Code", LocationCode);
    ItemLedgerEntry.CalcSums(Quantity);
    AvailableQty := ItemLedgerEntry.Quantity;

    exit(AvailableQty >= RequiredQty);
end;
```

## Bad Examples

### Unnecessary Nesting After Find
```al
procedure GetCustomerName(CustomerNo: Code[20]): Text[100]
var
    Customer: Record Customer;
begin
    if Customer.Get(CustomerNo) then begin
        // Unnecessary nesting - should return early on not found
        exit(Customer.Name);
    end else begin
        // Process all failure scenarios in else block
        if CustomerNo = '' then begin
            exit('Invalid Customer Number');
        end else begin
            exit('Customer Not Found');
        end;
    end;
end;
```

### Processing in Nested Blocks
```al
procedure ProcessCustomerOrders(CustomerNo: Code[20]): Boolean
var
    Customer: Record Customer;
    SalesHeader: Record "Sales Header";
    ProcessedCount: Integer;
begin
    if Customer.Get(CustomerNo) then begin
        if Customer.Blocked = Customer.Blocked::" " then begin
            SalesHeader.SetRange("Sell-to Customer No.", CustomerNo);
            SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
            if SalesHeader.FindSet() then begin
                repeat
                    if ProcessSalesOrder(SalesHeader."No.") then begin
                        ProcessedCount += 1;
                    end;
                until SalesHeader.Next() = 0;
                if ProcessedCount > 0 then begin
                    exit(true);
                end else begin
                    exit(false);
                end;
            end else begin
                exit(false);
            end;
        end else begin
            exit(false);
        end;
    end else begin
        exit(false);
    end;
end;
```

### Inefficient Multiple Lookups
```al
procedure GetItemDescription(ItemNo: Code[20]): Text[100]
var
    Item: Record Item;
    ItemTranslation: Record "Item Translation";
    Description: Text[100];
begin
    if Item.Get(ItemNo) then begin
        Description := Item.Description;
        // Inefficient: Could exit early with base description
        ItemTranslation.SetRange("Item No.", ItemNo);
        ItemTranslation.SetRange("Language Code", GlobalLanguage);
        if ItemTranslation.FindFirst() then begin
            if ItemTranslation.Description <> '' then begin
                Description := ItemTranslation.Description;
            end;
        end;
        exit(Description);
    end else begin
        exit('');
    end;
end;
```

## Best Practices

### Guard Clause Pattern
```al
procedure CalculateCustomerBalance(CustomerNo: Code[20]): Decimal
var
    Customer: Record Customer;
    CustLedgerEntry: Record "Cust. Ledger Entry";
begin
    // Guard clauses with early exits
    if CustomerNo = '' then
        exit(0);

    if not Customer.Get(CustomerNo) then
        exit(0);

    // Main logic only executes with valid data
    CustLedgerEntry.SetRange("Customer No.", CustomerNo);
    CustLedgerEntry.SetRange(Open, true);
    CustLedgerEntry.CalcSums("Remaining Amt. (LCY)");
    exit(CustLedgerEntry."Remaining Amt. (LCY)");
end;
```

### Efficient Existence Check
```al
procedure CustomerHasOpenOrders(CustomerNo: Code[20]): Boolean
var
    SalesHeader: Record "Sales Header";
begin
    // Early exit for invalid input
    if CustomerNo = '' then
        exit(false);

    // Efficient existence check with FindFirst
    SalesHeader.SetRange("Sell-to Customer No.", CustomerNo);
    SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
    SalesHeader.SetRange(Status, SalesHeader.Status::Open);

    // Return boolean directly - no additional processing needed
    exit(not SalesHeader.IsEmpty);
end;
```

### Optimized Filter Chain
```al
procedure GetActiveItemCount(ItemCategoryCode: Code[20]): Integer
var
    Item: Record Item;
begin
    // Early exit for empty category
    if ItemCategoryCode = '' then
        exit(0);

    // Set filters before find operation
    Item.SetRange("Item Category Code", ItemCategoryCode);
    Item.SetRange(Blocked, false);
    Item.SetRange(Type, Item.Type::Inventory);

    // Early exit if no matching records
    if Item.IsEmpty then
        exit(0);

    exit(Item.Count);
end;
```

### Resource Management Pattern
```al
procedure ProcessItemBatch(var ItemFilter: Record Item): Integer
var
    ProcessedCount: Integer;
    TempItem: Record Item temporary;
begin
    // Early exit if no records to process
    if ItemFilter.IsEmpty then
        exit(0);

    // Copy to temporary for safe processing
    if ItemFilter.FindSet() then begin
        repeat
            TempItem := ItemFilter;
            TempItem.Insert();
        until ItemFilter.Next() = 0;
    end;

    // Early exit if copy failed
    if TempItem.IsEmpty then
        exit(0);

    // Process temporary records
    TempItem.FindSet();
    repeat
        if ProcessSingleItem(TempItem) then
            ProcessedCount += 1;
    until TempItem.Next() = 0;

    exit(ProcessedCount);
end;
```

## Performance Guidelines

1. **Use early exits to avoid unnecessary database operations**
2. **Apply filters before Find operations to reduce record set**
3. **Check IsEmpty before iterating large record sets**
4. **Use FindFirst for existence checks instead of FindSet**
5. **Validate input parameters before database access**
6. **Group related validations for efficient early exits**