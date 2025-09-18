# AL Statement Per Line Rule - Code Examples

## Good Examples

### Single Statement Per Line
```al
procedure ProcessCustomerOrder(CustomerNo: Code[20])
var
    Customer: Record Customer;
    SalesHeader: Record "Sales Header";
    OrderNo: Code[20];
begin
    // Each statement on its own line for clarity
    Customer.Get(CustomerNo);
    Customer.CalcFields("Balance (LCY)");

    SalesHeader.Init();
    SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
    SalesHeader."Sell-to Customer No." := CustomerNo;
    SalesHeader."Order Date" := Today;
    SalesHeader.Insert(true);

    OrderNo := SalesHeader."No.";
    Message('Order %1 created successfully', OrderNo);
end;
```

### Variable Declarations - One Per Line
```al
procedure CalculateOrderTotals(OrderNo: Code[20])
var
    SalesHeader: Record "Sales Header";
    SalesLine: Record "Sales Line";
    TotalAmount: Decimal;
    TotalQuantity: Decimal;
    LineCount: Integer;
    DiscountAmount: Decimal;
begin
    SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo);

    SalesLine.SetRange("Document Type", SalesLine."Document Type"::Order);
    SalesLine.SetRange("Document No.", OrderNo);

    if SalesLine.FindSet() then
        repeat
            TotalAmount += SalesLine."Line Amount";
            TotalQuantity += SalesLine.Quantity;
            LineCount += 1;
            DiscountAmount += SalesLine."Line Discount Amount";
        until SalesLine.Next() = 0;

    SalesHeader."Total Amount" := TotalAmount;
    SalesHeader."Total Quantity" := TotalQuantity;
    SalesHeader.Modify();
end;
```

### Control Flow Statements
```al
procedure ValidateOrderData(var SalesHeader: Record "Sales Header"): Boolean
var
    Customer: Record Customer;
    ValidationResult: Boolean;
begin
    ValidationResult := true;

    if SalesHeader."Sell-to Customer No." = '' then
        ValidationResult := false;

    if ValidationResult then
        if not Customer.Get(SalesHeader."Sell-to Customer No.") then
            ValidationResult := false;

    if ValidationResult then
        if Customer.Blocked then
            ValidationResult := false;

    if ValidationResult then
        if SalesHeader."Order Date" = 0D then
            ValidationResult := false;

    exit(ValidationResult);
end;
```

### Complex Expressions Broken Down
```al
procedure CalculateComplexPricing(ItemNo: Code[20]; CustomerNo: Code[20]; Quantity: Decimal): Decimal
var
    Item: Record Item;
    Customer: Record Customer;
    BasePrice: Decimal;
    CustomerDiscount: Decimal;
    VolumeDiscount: Decimal;
    FinalPrice: Decimal;
begin
    Item.Get(ItemNo);
    Customer.Get(CustomerNo);

    BasePrice := Item."Unit Price";
    CustomerDiscount := GetCustomerDiscount(CustomerNo);
    VolumeDiscount := GetVolumeDiscount(Quantity);

    FinalPrice := BasePrice;
    FinalPrice := FinalPrice * (1 - CustomerDiscount / 100);
    FinalPrice := FinalPrice * (1 - VolumeDiscount / 100);

    exit(FinalPrice);
end;
```

## Bad Examples

### Multiple Statements Per Line
```al
procedure BadMultipleStatements(CustomerNo: Code[20])
var
    Customer: Record Customer; SalesHeader: Record "Sales Header"; OrderNo: Code[20];  // Bad: multiple declarations
begin
    Customer.Get(CustomerNo); Customer.CalcFields("Balance (LCY)"); SalesHeader.Init();  // Bad: multiple statements
    SalesHeader."Document Type" := SalesHeader."Document Type"::Order; SalesHeader."Sell-to Customer No." := CustomerNo; SalesHeader.Insert(true);  // Bad: cramped assignments
    OrderNo := SalesHeader."No."; Message('Order %1 created', OrderNo);  // Bad: multiple statements
end;
```

### Cramped Control Flow
```al
procedure BadControlFlow(OrderNo: Code[20])
var
    SalesLine: Record "Sales Line"; Total: Decimal;  // Bad: multiple declarations
begin
    SalesLine.SetRange("Document No.", OrderNo); if SalesLine.FindSet() then repeat Total += SalesLine."Line Amount"; until SalesLine.Next() = 0;  // Bad: entire loop on one line
    if Total > 1000 then Message('Large order') else Message('Small order');  // Bad: if-else on one line
end;
```

### Complex Expressions on Single Lines
```al
procedure BadComplexExpressions(ItemNo: Code[20]; CustomerNo: Code[20]; Qty: Decimal): Decimal
var
    Item: Record Item; Customer: Record Customer;  // Bad: multiple declarations
begin
    Item.Get(ItemNo); Customer.Get(CustomerNo);  // Bad: multiple statements
    exit(Item."Unit Price" * (1 - GetCustomerDiscount(CustomerNo) / 100) * (1 - GetVolumeDiscount(Qty) / 100) * Qty);  // Bad: complex calculation on one line
end;
```

### Nested Structures Cramped
```al
procedure BadNesting(OrderNo: Code[20])
var
    SalesLine: Record "Sales Line"; Item: Record Item;  // Bad: multiple declarations
begin
    SalesLine.SetRange("Document No.", OrderNo); if SalesLine.FindSet() then repeat if SalesLine.Type = SalesLine.Type::Item then if Item.Get(SalesLine."No.") then if Item.Inventory < SalesLine.Quantity then Error('Low inventory') else Item.Modify(); until SalesLine.Next() = 0;  // Bad: complex nested logic on one line
end;
```

### Variable Assignments and Calculations Cramped
```al
procedure BadCalculations()
var
    Price, Discount, Tax, Total: Decimal;  // Bad: multiple declarations on one line
    Qty1, Qty2, Qty3: Integer;  // Bad: multiple declarations
begin
    Price := 100; Discount := 0.1; Tax := 0.08; Qty1 := 5; Qty2 := 3; Qty3 := 2;  // Bad: multiple assignments
    Total := (Price * (Qty1 + Qty2 + Qty3)) * (1 - Discount) * (1 + Tax);  // Bad: complex calculation
    Message('Total: %1', Total); Exit(Total);  // Bad: multiple statements
end;
```

### Poor Error Handling Structure
```al
procedure BadErrorHandling(CustomerNo: Code[20])
var
    Customer: Record Customer; ErrorMsg: Text;  // Bad: multiple declarations
begin
    if not Customer.Get(CustomerNo) then ErrorMsg := 'Customer not found'; if Customer.Blocked then ErrorMsg += ' and blocked'; if ErrorMsg <> '' then Error(ErrorMsg);  // Bad: complex error logic on one line
end;
```

## Best Practices

1. **One statement per line** - each operation should have its own line
2. **One variable declaration per line** - easier to read and maintain
3. **Break complex expressions** into multiple assignment statements
4. **Separate logical operations** even if they could be combined
5. **Use meaningful intermediate variables** for complex calculations
6. **Structure control flow** with proper line breaks and indentation