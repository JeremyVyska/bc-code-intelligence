# AL Comment Spacing Standards - Code Examples

## Good Examples

### Single-Line Comments with Proper Spacing
```al
procedure ProcessCustomerOrder(CustomerNo: Code[20])
var
    Customer: Record Customer;
    SalesHeader: Record "Sales Header";
begin
    // Validate customer exists and is not blocked
    if not Customer.Get(CustomerNo) then
        Error('Customer %1 does not exist', CustomerNo);

    // Check customer credit limit before processing
    Customer.CalcFields("Balance (LCY)");
    if Customer."Credit Limit (LCY)" < Customer."Balance (LCY)" then
        Error('Customer %1 has exceeded credit limit', CustomerNo);

    // Create new sales order
    SalesHeader.Init();
    SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
    SalesHeader."Sell-to Customer No." := CustomerNo;
    SalesHeader.Insert(true);
end;
```

### Multi-Line Comments with Consistent Formatting
```al
codeunit 50100 "Order Processing"
{
    /*
     * ProcessBulkOrders procedure handles multiple order processing
     * with validation and error handling for each order in the batch.
     * Returns count of successfully processed orders.
     */
    procedure ProcessBulkOrders(var OrderList: List of [Code[20]]): Integer
    var
        ProcessedCount: Integer;
        OrderNo: Code[20];
    begin
        ProcessedCount := 0;

        /*
         * Iterate through each order in the list
         * and attempt to process with error recovery
         */
        foreach OrderNo in OrderList do begin
            if TryProcessSingleOrder(OrderNo) then
                ProcessedCount += 1;
        end;

        exit(ProcessedCount);
    end;
}
```

### Inline Comments with Proper Spacing
```al
procedure CalculateDiscount(Amount: Decimal; CustomerType: Option Regular,Premium,VIP): Decimal
var
    DiscountPct: Decimal;
begin
    case CustomerType of
        CustomerType::Regular:
            DiscountPct := 0.05;    // 5% discount for regular customers
        CustomerType::Premium:
            DiscountPct := 0.10;    // 10% discount for premium customers
        CustomerType::VIP:
            DiscountPct := 0.15;    // 15% discount for VIP customers
    end;

    exit(Amount * DiscountPct);     // Calculate final discount amount
end;
```

### Section Headers with Consistent Spacing
```al
codeunit 50101 "Customer Management"
{
    // ============================================================================
    // Validation Functions
    // ============================================================================

    procedure ValidateCustomerData(var Customer: Record Customer): Boolean
    begin
        // Implementation here
    end;

    procedure ValidateAddressInfo(var Customer: Record Customer): Boolean
    begin
        // Implementation here
    end;

    // ============================================================================
    // Processing Functions
    // ============================================================================

    procedure CreateCustomer(Name: Text[100]): Code[20]
    begin
        // Implementation here
    end;

    procedure UpdateCustomer(var Customer: Record Customer)
    begin
        // Implementation here
    end;
}
```

## Bad Examples

### Inconsistent Comment Spacing
```al
procedure BadSpacing(CustomerNo: Code[20])
var
    Customer: Record Customer;
begin
    //No space after comment marker
    if not Customer.Get(CustomerNo) then
        Error('Customer not found');

    //    Too many spaces after marker
    Customer.CalcFields("Balance (LCY)");

    // Good spacing
    if Customer.Blocked then
        Error('Customer is blocked');

    //Another comment without space
    Customer.Modify();
end;
```

### Poor Multi-Line Comment Formatting
```al
/*
Poorly formatted multi-line comment
without consistent indentation
    or proper spacing
        making it hard to read
*/
procedure PoorCommentFormatting()
begin
    /*This comment is not properly
    formatted and lacks
        consistent spacing*/
    Message('Hello');
end;
```

### Cramped Inline Comments
```al
procedure CrampedComments(Amount: Decimal): Decimal
var
    Tax: Decimal;
begin
    Tax := Amount * 0.08;//No space before comment
    Amount := Amount + Tax;//Another cramped comment
    exit(Amount);//Final cramped comment
end;
```

### Missing Comment Spacing in Complex Code
```al
procedure NoCommentSpacing()
var
    i: Integer;
    Total: Decimal;
begin
    for i := 1 to 10 do begin//Loop comment crammed
        if i mod 2 = 0 then//Even number check crammed
            Total += i * 2;//Double even numbers crammed
    end;//End loop comment crammed
end;
```

### Inconsistent Section Spacing
```al
codeunit 50102 "Poor Comment Organization"
{
    //=== Validation Functions ===
    procedure ValidateData(): Boolean
    begin
        // Implementation
    end;

    // Processing Functions (inconsistent header style)
    procedure ProcessData()
    begin
        // Implementation
    end;

    //===Utility Functions===(no spaces around text)
    procedure UtilityFunction()
    begin
        // Implementation
    end;
}
```

## Best Practices

1. **Always use single space** after // for single-line comments
2. **Consistent indentation** for multi-line comments with /* */
3. **Appropriate spacing** before inline comments (at least 4 spaces)
4. **Uniform section headers** with consistent formatting style
5. **Group related comments** with consistent spacing patterns