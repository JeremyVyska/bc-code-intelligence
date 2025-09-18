# AL Separate If-Else Formatting - Code Examples

## Good Examples

### Proper If-Else Separation with Clear Structure
```al
procedure ProcessCustomerStatus(CustomerNo: Code[20])
var
    Customer: Record Customer;
begin
    if Customer.Get(CustomerNo) then
    begin
        if not Customer.Blocked then
        begin
            Customer."Last Contact Date" := Today;
            Customer.Modify();
            Message('Customer %1 updated successfully', CustomerNo);
        end
        else
        begin
            Message('Customer %1 is blocked and cannot be updated', CustomerNo);
        end;
    end
    else
    begin
        Error('Customer %1 does not exist', CustomerNo);
    end;
end;
```

### Multiple If-Else Conditions with Proper Separation
```al
procedure HandleOrderStatus(var SalesHeader: Record "Sales Header")
begin
    if SalesHeader.Status = SalesHeader.Status::Open then
    begin
        EnableEditingActions();
        SetStatusColor('Green');
        ShowMessage('Order is open for editing');
    end
    else if SalesHeader.Status = SalesHeader.Status::Released then
    begin
        DisableEditingActions();
        SetStatusColor('Blue');
        ShowMessage('Order has been released');
    end
    else if SalesHeader.Status = SalesHeader.Status::"Pending Approval" then
    begin
        DisableEditingActions();
        SetStatusColor('Yellow');
        ShowMessage('Order is pending approval');
        RequestApproval(SalesHeader);
    end
    else
    begin
        DisableAllActions();
        SetStatusColor('Red');
        ShowMessage('Order status is invalid');
    end;
end;
```

### Nested If-Else with Clear Block Structure
```al
procedure ValidateAndProcessOrder(OrderNo: Code[20])
var
    SalesHeader: Record "Sales Header";
    Customer: Record Customer;
begin
    if SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo) then
    begin
        if Customer.Get(SalesHeader."Sell-to Customer No.") then
        begin
            if not Customer.Blocked then
            begin
                if SalesHeader."Order Date" <> 0D then
                begin
                    ProcessValidOrder(SalesHeader);
                    Message('Order processed successfully');
                end
                else
                begin
                    Error('Order date is required');
                end;
            end
            else
            begin
                Error('Customer %1 is blocked', Customer."No.");
            end;
        end
        else
        begin
            Error('Customer does not exist');
        end;
    end
    else
    begin
        Error('Order %1 not found', OrderNo);
    end;
end;
```

### Complex Business Logic with Proper If-Else Structure
```al
procedure CalculateDiscount(CustomerNo: Code[20]; OrderAmount: Decimal): Decimal
var
    Customer: Record Customer;
    DiscountPct: Decimal;
begin
    if Customer.Get(CustomerNo) then
    begin
        if Customer."Customer Posting Group" = 'VIP' then
        begin
            if OrderAmount >= 10000 then
            begin
                DiscountPct := 15;
            end
            else if OrderAmount >= 5000 then
            begin
                DiscountPct := 10;
            end
            else
            begin
                DiscountPct := 5;
            end;
        end
        else if Customer."Customer Posting Group" = 'PREMIUM' then
        begin
            if OrderAmount >= 5000 then
            begin
                DiscountPct := 8;
            end
            else
            begin
                DiscountPct := 3;
            end;
        end
        else
        begin
            if OrderAmount >= 1000 then
            begin
                DiscountPct := 2;
            end
            else
            begin
                DiscountPct := 0;
            end;
        end;
    end
    else
    begin
        DiscountPct := 0;
    end;

    exit(DiscountPct);
end;
```

## Bad Examples

### Cramped If-Else on Same Lines
```al
procedure BadIfElseFormatting(CustomerNo: Code[20])
var
    Customer: Record Customer;
begin
    if Customer.Get(CustomerNo) then begin Customer."Last Contact Date" := Today; Customer.Modify(); end else begin Error('Customer not found'); end;  // Bad: entire if-else on one line

    if not Customer.Blocked then Message('Active customer') else Message('Blocked customer');  // Bad: no begin-end blocks, single line
end;
```

### Poor Else Positioning
```al
procedure PoorElsePositioning(OrderNo: Code[20])
var
    SalesHeader: Record "Sales Header";
begin
    if SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo) then
    begin
        ProcessOrder(SalesHeader);
    end else begin  // Bad: else on same line as end
        Error('Order not found');
    end;

    if SalesHeader.Status = SalesHeader.Status::Open then
    begin
        EnableEditing();
    end
        else  // Bad: else indented incorrectly
    begin
        DisableEditing();
    end;
end;
```

### Missing Begin-End Blocks
```al
procedure MissingBeginEnd(CustomerNo: Code[20])
var
    Customer: Record Customer;
begin
    if Customer.Get(CustomerNo) then
        Customer."Last Contact Date" := Today;
        Customer.Modify();  // Bad: this executes regardless of if condition
    else
        Error('Customer not found');  // Bad: this causes compilation error

    if Customer.Blocked then
        Customer.Blocked := false;
        Message('Customer unblocked');  // Bad: this executes regardless of if condition
    else
        Message('Customer is active');
end;
```

### Inconsistent Formatting Styles
```al
procedure InconsistentFormatting(OrderNo: Code[20])
var
    SalesHeader: Record "Sales Header";
    Customer: Record Customer;
begin
    // Style 1: proper formatting
    if SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo) then
    begin
        ProcessOrder(SalesHeader);
    end
    else
    begin
        Error('Order not found');
    end;

    // Style 2: cramped formatting
    if Customer.Get(SalesHeader."Sell-to Customer No.") then begin ProcessCustomer(Customer); end else begin Error('Customer not found'); end;

    // Style 3: missing blocks
    if SalesHeader."Order Date" = Today then
        Message('Today order')
    else
        Message('Past order');
end;
```

### Complex Nested Logic Without Clear Structure
```al
procedure PoorNestedStructure(OrderNo: Code[20])
var
    SalesHeader: Record "Sales Header";
    Customer: Record Customer;
begin
    if SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo) then if Customer.Get(SalesHeader."Sell-to Customer No.") then if not Customer.Blocked then if SalesHeader."Order Date" <> 0D then ProcessValidOrder(SalesHeader) else Error('No order date') else Error('Customer blocked') else Error('Customer not found') else Error('Order not found');  // Bad: entire nested structure on one line
end;
```

### Poor Else-If Chain Formatting
```al
procedure PoorElseIfChain(Status: Option Open,Released,Pending,Cancelled)
begin
    if Status = Status::Open then begin
        EnableEditing();
    end else if Status = Status::Released then begin  // Bad: else if on same line as end
        DisableEditing();
    end
    else if Status = Status::Pending then  // Bad: inconsistent else positioning
    begin
        RequestApproval();
    end else begin  // Bad: else on same line as end
        ShowError();
    end;
end;
```

## Best Practices

1. **Separate if and else blocks** with proper line breaks
2. **Use begin-end blocks** for all multi-statement conditions
3. **Consistent else positioning** - always on its own line aligned with corresponding if
4. **Clear visual hierarchy** through proper indentation
5. **Avoid cramping** multiple conditions or statements on single lines