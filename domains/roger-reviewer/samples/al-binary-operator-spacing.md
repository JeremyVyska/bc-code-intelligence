# AL Binary Operator Spacing - Code Examples

## Good Examples

### Arithmetic Operators with Proper Spacing
```al
procedure CalculateOrderTotals(BaseAmount: Decimal; TaxRate: Decimal; DiscountPct: Decimal): Decimal
var
    TaxAmount: Decimal;
    DiscountAmount: Decimal;
    NetAmount: Decimal;
    FinalTotal: Decimal;
begin
    // Arithmetic operations with consistent spacing
    TaxAmount := BaseAmount * TaxRate / 100;
    DiscountAmount := BaseAmount * DiscountPct / 100;
    NetAmount := BaseAmount - DiscountAmount;
    FinalTotal := NetAmount + TaxAmount;

    // Complex calculations with proper spacing
    FinalTotal := (BaseAmount * (1 - DiscountPct / 100)) * (1 + TaxRate / 100);

    exit(FinalTotal);
end;
```

### Comparison Operators with Consistent Spacing
```al
procedure ValidateCustomerData(var Customer: Record Customer): Boolean
var
    ValidationResult: Boolean;
begin
    ValidationResult := true;

    // Comparison operations with proper spacing
    if Customer."No." = '' then
        ValidationResult := false;

    if Customer."Credit Limit (LCY)" <= 0 then
        ValidationResult := false;

    if Customer."Balance (LCY)" >= Customer."Credit Limit (LCY)" then
        ValidationResult := false;

    if Customer."Last Date Modified" <> Today then
        Customer."Last Date Modified" := Today;

    // Complex comparison with proper spacing
    if (Customer."Balance (LCY)" > 0) and (Customer."Credit Limit (LCY)" > 0) then
        if Customer."Balance (LCY)" / Customer."Credit Limit (LCY)" > 0.8 then
            Message('Customer approaching credit limit');

    exit(ValidationResult);
end;
```

### Logical Operators with Clear Spacing
```al
procedure ProcessOrderLogic(var SalesHeader: Record "Sales Header"): Boolean
var
    CanProcess: Boolean;
    Customer: Record Customer;
begin
    Customer.Get(SalesHeader."Sell-to Customer No.");

    // Logical operations with proper spacing
    CanProcess := (SalesHeader.Status = SalesHeader.Status::Open) and
                  (not Customer.Blocked) and
                  (SalesHeader."Order Date" <> 0D);

    // Complex logical expressions with clear spacing
    if (Customer."Credit Limit (LCY)" > 0) and
       (Customer."Balance (LCY)" < Customer."Credit Limit (LCY)") and
       ((SalesHeader."Amount Including VAT" + Customer."Balance (LCY)") <= Customer."Credit Limit (LCY)") then
        CanProcess := true
    else
        CanProcess := false;

    // Multiple condition checks with proper spacing
    if (SalesHeader."Shipment Date" <> 0D) or
       (SalesHeader."Requested Delivery Date" <> 0D) or
       (SalesHeader."Promised Delivery Date" <> 0D) then
        SalesHeader."Has Delivery Date" := true;

    exit(CanProcess);
end;
```

### Assignment Operators with Consistent Formatting
```al
procedure UpdateCustomerInformation(var Customer: Record Customer; NewCredit: Decimal)
var
    PreviousCredit: Decimal;
    CreditIncrease: Decimal;
begin
    // Assignment operations with proper spacing
    PreviousCredit := Customer."Credit Limit (LCY)";
    Customer."Credit Limit (LCY)" := NewCredit;
    CreditIncrease := NewCredit - PreviousCredit;

    // Complex assignments with proper spacing
    Customer."Credit Available (LCY)" := Customer."Credit Limit (LCY)" - Customer."Balance (LCY)";

    // Multiple assignments maintaining consistency
    Customer."Last Date Modified" := Today;
    Customer."Last Time Modified" := Time;
    Customer."Modified By" := UserId;

    Customer.Modify();
end;
```

### Mixed Operators in Complex Expressions
```al
procedure CalculateComplexPricing(ItemNo: Code[20]; Quantity: Decimal; CustomerNo: Code[20]): Decimal
var
    Item: Record Item;
    Customer: Record Customer;
    BasePrice: Decimal;
    VolumeDiscount: Decimal;
    CustomerDiscount: Decimal;
    FinalPrice: Decimal;
begin
    Item.Get(ItemNo);
    Customer.Get(CustomerNo);

    BasePrice := Item."Unit Price";

    // Complex calculations with consistent operator spacing
    if Quantity >= 100 then
        VolumeDiscount := 0.15
    else if Quantity >= 50 then
        VolumeDiscount := 0.10
    else if Quantity >= 20 then
        VolumeDiscount := 0.05
    else
        VolumeDiscount := 0.00;

    // Customer type discount calculation
    if Customer."Customer Price Group" = 'VIP' then
        CustomerDiscount := 0.20
    else if Customer."Customer Price Group" = 'PREMIUM' then
        CustomerDiscount := 0.10
    else
        CustomerDiscount := 0.00;

    // Final price calculation with proper spacing
    FinalPrice := BasePrice * Quantity * (1 - VolumeDiscount) * (1 - CustomerDiscount);

    exit(FinalPrice);
end;
```

## Bad Examples

### Cramped Arithmetic Operations
```al
procedure BadArithmeticSpacing(Amount: Decimal; Tax: Decimal): Decimal
var
    Result: Decimal;
begin
    Result:=Amount*Tax/100;  // Bad: no spaces around operators
    Result:=Amount+Result;   // Bad: no spaces around operators
    Result:=(Amount*1.1)+Tax-5;  // Bad: inconsistent spacing
    exit(Result);
end;
```

### Inconsistent Comparison Spacing
```al
procedure BadComparisonSpacing(Customer: Record Customer): Boolean
begin
    if Customer."No."='' then  // Bad: no spaces around =
        exit(false);

    if Customer."Credit Limit (LCY)" <=0 then  // Bad: no space before <=
        exit(false);

    if Customer."Balance (LCY)">=  Customer."Credit Limit (LCY)" then  // Bad: inconsistent spacing around >=
        exit(false);

    exit(true);
end;
```

### Poor Logical Operator Spacing
```al
procedure BadLogicalSpacing(SalesHeader: Record "Sales Header"): Boolean
var
    Customer: Record Customer;
begin
    Customer.Get(SalesHeader."Sell-to Customer No.");

    // Bad logical operator spacing
    if(SalesHeader.Status=SalesHeader.Status::Open)and(not Customer.Blocked)and(SalesHeader."Order Date"<>0D)then  // Bad: no spaces
        exit(true);

    if (Customer."Credit Limit (LCY)">0)and (Customer."Balance (LCY)"<Customer."Credit Limit (LCY)")then  // Bad: inconsistent spacing
        exit(true);

    exit(false);
end;
```

### Cramped Assignment Operations
```al
procedure BadAssignmentSpacing(var Customer: Record Customer)
var
    NewCredit: Decimal;
begin
    NewCredit:=5000;  // Bad: no spaces around :=
    Customer."Credit Limit (LCY)":=NewCredit;  // Bad: no spaces around :=
    Customer."Credit Available (LCY)":=Customer."Credit Limit (LCY)"-Customer."Balance (LCY)";  // Bad: no spaces around operators
end;
```

### Mixed Spacing Inconsistency
```al
procedure InconsistentSpacing(Amount: Decimal; Discount: Decimal): Decimal
var
    Result: Decimal;
begin
    Result := Amount * (1-Discount/100);  // Bad: inconsistent spacing (some operators have spaces, others don't)
    Result:= Result+10;  // Bad: space only on one side of :=
    Result=Result *1.1;  // Bad: wrong assignment operator and poor spacing

    if Result> 1000 then  // Bad: space only on one side of >
        Result := Result- 50;  // Bad: space only on one side of -

    exit(Result);
end;
```

### Extremely Cramped Complex Expressions
```al
procedure CrampedComplexExpressions(Item: Record Item; Qty: Decimal; Customer: Record Customer): Decimal
var
    Price: Decimal;
begin
    Price:=Item."Unit Price"*Qty*(1-GetVolumeDiscount(Qty)/100)*(1-GetCustomerDiscount(Customer."No.")/100);  // Bad: no spacing in complex expression

    if(Item."Unit Price">0)and(Qty>0)and(Customer."Credit Limit (LCY)">Customer."Balance (LCY)")then  // Bad: cramped logical expression
        Price:=Price*1.1;  // Bad: no spaces

    exit(Price);
end;
```

## Best Practices

1. **Single space before and after** all binary operators (+, -, *, /, =, <>, <, >, <=, >=, :=)
2. **Consistent spacing** throughout all expressions and assignments
3. **Clear separation** in complex expressions with parentheses and spacing
4. **Maintain spacing** in multi-line expressions for readability
5. **Use consistent patterns** across entire codebase for professional appearance