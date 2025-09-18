# Begin/End Positioning - AL Code Examples

## Begin as Afterword Pattern

```al
// Recommended: Begin as afterword pattern
procedure ProcessCustomerData(CustomerNo: Code[20]) begin
    if Customer.Get(CustomerNo) then begin
        ValidateCustomerData();
        UpdateCustomerStatus();
        LogCustomerAccess();
    end;
end;

// Alternative: Begin on separate line
procedure ProcessCustomerDataAlternative(CustomerNo: Code[20])
begin
    if Customer.Get(CustomerNo) then
    begin
        ValidateCustomerData();
        UpdateCustomerStatus();
        LogCustomerAccess();
    end;
end;
```

## Consistent Indentation Patterns

```al
// Good: Consistent 4-space indentation
procedure ProcessSalesOrder(OrderNo: Code[20]) begin
    if SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo) then begin
        if ValidateOrderData() then begin
            if ProcessOrderLines() then begin
                UpdateOrderStatus();
                SendConfirmationEmail();
            end else begin
                LogProcessingError();
                RollbackChanges();
            end;
        end;
    end;
end;

// Poor: Inconsistent indentation
procedure ProcessSalesOrderBad(OrderNo: Code[20]) begin
if SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo) then begin
  if ValidateOrderData() then begin
        if ProcessOrderLines() then begin
    UpdateOrderStatus();
            SendConfirmationEmail();
        end else begin
LogProcessingError();
      RollbackChanges();
        end;
  end;
end;
end;
```

## Trigger Formatting Examples

```al
// Table trigger with begin as afterword
table 50100 "Customer Extended"
{
    fields
    {
        field(1; "No."; Code[20]) begin
        end;

        field(2; Name; Text[100]) begin
            trigger OnValidate() begin
                if Name <> '' then begin
                    UpdateSearchName();
                    LogNameChange();
                end;
            end;
        end;
    }

    trigger OnInsert() begin
        TestField("No.");
        if "No." = '' then begin
            GetNextNumber();
        end;
        InitializeDefaults();
    end;

    trigger OnModify() begin
        if xRec.Name <> Name then begin
            ValidateNameChange();
            UpdateRelatedRecords();
        end;
    end;
}
```

## Control Structure Positioning

```al
// Consistent if-then-else formatting
procedure DetermineShippingMethod(Amount: Decimal; Country: Code[10]): Code[10] begin
    if Country = 'US' then begin
        if Amount > 1000 then begin
            exit('PREMIUM-DOMESTIC');
        end else begin
            exit('STANDARD-DOMESTIC');
        end;
    end else begin
        if Amount > 2000 then begin
            exit('PREMIUM-INTERNATIONAL');
        end else begin
            exit('STANDARD-INTERNATIONAL');
        end;
    end;
end;

// Case statement formatting
procedure ProcessOrderByType(OrderType: Option Standard,DropShip,Transfer) begin
    case OrderType of
        OrderType::Standard: begin
            ProcessStandardOrder();
            UpdateInventory();
        end;
        OrderType::DropShip: begin
            CreatePurchaseOrder();
            NotifyVendor();
        end;
        OrderType::Transfer: begin
            ValidateLocations();
            CreateTransferOrder();
        end;
        else begin
            Error('Unknown order type');
        end;
    end;
end;
```

## Professional Formatting Standards

```al
// Professional procedure organization
codeunit 50100 "Order Processing Manager"
{
    var
        Customer: Record Customer;
        SalesHeader: Record "Sales Header";

    procedure CreateSalesOrder(CustomerNo: Code[20]; OrderDate: Date): Code[20] begin
        ValidateCustomer(CustomerNo);
        InitializeOrder(CustomerNo, OrderDate);
        exit(SalesHeader."No.");
    end;

    local procedure ValidateCustomer(CustomerNo: Code[20]) begin
        if not Customer.Get(CustomerNo) then begin
            Error('Customer %1 does not exist', CustomerNo);
        end;

        if Customer.Blocked <> Customer.Blocked::" " then begin
            Error('Customer %1 is blocked', CustomerNo);
        end;

        Customer.CheckCreditLimit(0, true);
    end;

    local procedure InitializeOrder(CustomerNo: Code[20]; OrderDate: Date) begin
        SalesHeader.Init();
        SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
        SalesHeader."No." := GetNextOrderNumber();
        SalesHeader."Sell-to Customer No." := CustomerNo;
        SalesHeader."Order Date" := OrderDate;
        SalesHeader.Insert(true);

        if SalesHeader."Bill-to Customer No." = '' then begin
            SalesHeader."Bill-to Customer No." := CustomerNo;
            SalesHeader.Modify(true);
        end;
    end;
}
```

## Anti-Pattern Examples

```al
// Anti-pattern: Inconsistent begin positioning
procedure BadFormattingExample() begin
if Customer.FindFirst() then
begin  // Inconsistent - should match procedure style
    repeat
        if Customer.Blocked = Customer.Blocked::" "
        then begin  // Awkward line break
            ProcessCustomer();
        end
        else
        begin  // Inconsistent with above
            SkipCustomer();
        end;
    until Customer.Next() = 0;
end;
end;

// Anti-pattern: Mixed indentation styles
procedure MixedIndentationBad() begin
    if SomeCondition then begin
      DoSomething();  // 2 spaces
        DoSomethingElse();  // 4 spaces
	DoAnother();  // Tab character
    end;
end;

// Better: Consistent formatting
procedure ConsistentFormattingGood() begin
    if SomeCondition then begin
        DoSomething();
        DoSomethingElse();
        DoAnother();
    end;
end;
```

## Team Standards Implementation

```al
// Example team standard: Begin as afterword with 4-space indentation
codeunit 50200 "Team Standard Example"
{
    procedure TeamStandardProcedure(Parameter: Text) begin
        if Parameter <> '' then begin
            ProcessParameter(Parameter);
            LogProcessing();
        end else begin
            HandleEmptyParameter();
        end;
    end;

    local procedure ProcessParameter(Param: Text) begin
        // Consistent formatting throughout codebase
        ValidateParameter(Param);
        TransformParameter(Param);
        SaveParameter(Param);
    end;

    local procedure HandleEmptyParameter() begin
        Message('Parameter cannot be empty');
    end;
}
```

## Best Practices Summary

### Formatting Guidelines
1. **Choose one style** and apply consistently across entire codebase
2. **Use 4-space indentation** for each nesting level
3. **Align begin/end keywords** at same indentation level
4. **Configure development tools** for automatic formatting
5. **Document team standards** for new team members

### Readability Optimization
1. **Maintain visual hierarchy** through consistent indentation
2. **Group related code** within logical begin/end blocks
3. **Use whitespace effectively** to separate logical sections
4. **Keep nesting reasonable** - avoid excessive depth
5. **Consider screen space** when choosing vertical vs horizontal formatting

### Team Collaboration
1. **Establish formatting standards** before starting projects
2. **Use code formatting tools** to enforce consistency
3. **Review formatting** as part of code review process
4. **Train team members** on adopted standards
5. **Update standards document** when making changes

Related Resources:
- AL code formatting guide: al-formatting-standards.md
- Team development standards: team-coding-guidelines.md
- Code review checklist: code-review-formatting.md