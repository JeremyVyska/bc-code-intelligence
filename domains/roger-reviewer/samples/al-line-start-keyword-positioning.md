# AL Line Start Keyword Positioning - Code Examples

## Good Examples

### Proper Keyword Line Positioning
```al
procedure ProcessCustomerOrder(CustomerNo: Code[20])
var
    Customer: Record Customer;
    SalesHeader: Record "Sales Header";
    OrderCreated: Boolean;
begin
    // Each keyword starts at appropriate line position
    if Customer.Get(CustomerNo) then
    begin
        if not Customer.Blocked then
        begin
            SalesHeader.Init();
            SalesHeader."Sell-to Customer No." := CustomerNo;

            if SalesHeader.Insert(true) then
            begin
                OrderCreated := true;
                Message('Order created successfully');
            end
            else
            begin
                OrderCreated := false;
                Error('Failed to create order');
            end;
        end
        else
        begin
            Error('Customer %1 is blocked', CustomerNo);
        end;
    end
    else
    begin
        Error('Customer %1 does not exist', CustomerNo);
    end;
end;
```

### Control Flow Keyword Positioning
```al
procedure ValidateOrderData(var SalesHeader: Record "Sales Header"): Boolean
var
    Customer: Record Customer;
    ValidationPassed: Boolean;
begin
    ValidationPassed := true;

    // Proper if-then-else positioning
    if SalesHeader."Sell-to Customer No." = '' then
    begin
        ValidationPassed := false;
        Error('Customer number is required');
    end;

    // Proper case statement positioning
    case SalesHeader."Document Type" of
        SalesHeader."Document Type"::Quote:
        begin
            if SalesHeader."Quote Valid Until Date" = 0D then
            begin
                ValidationPassed := false;
                Error('Quote validity date is required');
            end;
        end;
        SalesHeader."Document Type"::Order:
        begin
            if SalesHeader."Requested Delivery Date" = 0D then
            begin
                ValidationPassed := false;
                Error('Delivery date is required');
            end;
        end;
    end;

    // Proper while loop positioning
    while Customer.Next() <> 0 do
    begin
        if Customer.Blocked then
        begin
            Customer.Blocked := false;
            Customer.Modify();
        end;
    end;

    exit(ValidationPassed);
end;
```

### Function Declaration Positioning
```al
codeunit 50100 "Order Management"
{
    procedure CreateOrder(CustomerNo: Code[20]): Code[20]
    var
        SalesHeader: Record "Sales Header";
    begin
        // Implementation
        exit(SalesHeader."No.");
    end;

    procedure UpdateOrder(OrderNo: Code[20]; NewDate: Date)
    var
        SalesHeader: Record "Sales Header";
    begin
        // Implementation
    end;

    local procedure ValidateOrder(var SalesHeader: Record "Sales Header"): Boolean
    var
        ValidationResult: Boolean;
    begin
        // Implementation
        exit(ValidationResult);
    end;

    trigger OnRun()
    begin
        // Implementation
    end;
}
```

## Bad Examples

### Poor Keyword Line Positioning
```al
procedure BadKeywordPositioning(CustomerNo: Code[20])
var Customer: Record Customer; SalesHeader: Record "Sales Header";  // Wrong: multiple declarations on one line
begin if Customer.Get(CustomerNo) then begin  // Wrong: multiple keywords on one line
        if not Customer.Blocked then begin SalesHeader.Init(); SalesHeader."Sell-to Customer No." := CustomerNo;  // Wrong: cramped formatting
            if SalesHeader.Insert(true) then begin Message('Success'); end else begin Error('Failed'); end;  // Wrong: entire conditional on one line
        end else begin Error('Customer blocked'); end;  // Wrong: else statement positioning
    end else begin Error('Customer not found'); end;  // Wrong: else statement positioning
end;
```

### Inconsistent Indentation and Positioning
```al
procedure InconsistentPositioning(OrderNo: Code[20])
var
    SalesLine: Record "Sales Line";
    Item: Record Item;
begin
SalesLine.SetRange("Document No.", OrderNo);  // Wrong: not indented

    if SalesLine.FindSet() then  // Wrong: over-indented
repeat  // Wrong: should be on new line with proper indentation
        if SalesLine.Type = SalesLine.Type::Item then  // Wrong: inconsistent indentation
begin  // Wrong: should align with if
    Item.Get(SalesLine."No.");  // Wrong: poor indentation
        Item.Inventory -= SalesLine.Quantity;  // Wrong: inconsistent with previous line
end  // Wrong: missing semicolon and poor positioning
    else begin  // Wrong: else positioning and indentation
SalesLine."Unit Price" := 0;  // Wrong: inconsistent indentation
        end;  // Wrong: inconsistent with begin
until SalesLine.Next() = 0;  // Wrong: poor positioning
end;
```

### Cramped Control Structures
```al
procedure CrampedStructures(Status: Option Open,Released,Pending)
begin
    case Status of Status::Open: begin Message('Open'); EnableEditing(); end; Status::Released: begin Message('Released'); DisableEditing(); end; Status::Pending: begin Message('Pending'); RequestApproval(); end; end;  // Wrong: entire case statement on one line

    if Status = Status::Open then begin EnableEditing(); SetStatusColor('Green'); end else if Status = Status::Released then begin DisableEditing(); SetStatusColor('Blue'); end else begin DisableEditing(); SetStatusColor('Red'); end;  // Wrong: complex conditional on one line
end;
```

### Poor Function Declaration Formatting
```al
codeunit 50101 "Bad Formatting"
{
    procedure CreateCustomer(Name: Text[100]; Email: Text[80]; Phone: Text[30]): Code[20] var Customer: Record Customer; CustomerNo: Code[20]; begin Customer.Init(); CustomerNo := NoSeriesMgt.GetNextNo(Setup."Customer Nos.", 0D, true); Customer."No." := CustomerNo; Customer.Name := Name; exit(CustomerNo); end;  // Wrong: entire procedure on few lines

    local procedure ValidateData(Name: Text[100]): Boolean begin if Name = '' then exit(false); exit(true); end;  // Wrong: procedure body on declaration line

trigger OnRun() begin Message('Running'); end;  // Wrong: trigger not properly positioned
}
```

### Mixed Line Positioning Styles
```al
procedure MixedStyles(OrderNo: Code[20])
var
    SalesHeader: Record "Sales Header";
    ProcessingResult: Boolean;
begin
    if SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo) then begin  // Style 1: keywords on same line
        ProcessingResult := true;
        Message('Order found');
    end
    else  // Style 2: else on separate line
    begin
        ProcessingResult := false;
        Error('Order not found');
    end;

    case SalesHeader.Status of  // Style 3: case body on same line
        SalesHeader.Status::Open: ProcessingResult := true;
        SalesHeader.Status::Released:  // Style 4: case action on new line
            ProcessingResult := false;
    end;
end;
```

## Best Practices

1. **Start each major keyword** (if, case, while, for) on its own line
2. **Consistent indentation** for all keywords at the same logical level
3. **Align related keywords** (begin/end pairs, if/then/else structures)
4. **Separate complex logic** across multiple lines for readability
5. **Maintain consistent style** throughout the entire codeunit