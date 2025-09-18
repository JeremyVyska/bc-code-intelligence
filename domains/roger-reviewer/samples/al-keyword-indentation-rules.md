# AL Keyword Indentation Rules - Code Examples

## Good Examples

### Proper Trigger Indentation
```al
table 50100 "Customer Extension"
{
    fields
    {
        field(1; "No."; Code[20])
        {
            Caption = 'No.';
        }
        field(2; Name; Text[100])
        {
            Caption = 'Name';
        }
    }

    trigger OnInsert()
    begin
        if "No." = '' then
            "No." := NoSeriesMgt.GetNextNo(Setup."Customer Nos.", 0D, true);
    end;

    trigger OnModify()
    begin
        "Last Modified Date" := Today;
        "Last Modified Time" := Time;
    end;
}
```

### Procedure and Function Indentation
```al
codeunit 50100 "Customer Management"
{
    procedure CreateCustomer(Name: Text[100]; Email: Text[80]): Code[20]
    var
        Customer: Record Customer;
        CustomerNo: Code[20];
    begin
        // Validate input parameters
        if Name = '' then
            Error('Customer name cannot be empty');

        // Create customer record
        Customer.Init();
        CustomerNo := NoSeriesMgt.GetNextNo(SalesSetup."Customer Nos.", 0D, true);
        Customer."No." := CustomerNo;
        Customer.Name := Name;
        Customer."E-Mail" := Email;
        Customer.Insert(true);

        exit(CustomerNo);
    end;

    local procedure ValidateCustomerData(var Customer: Record Customer)
    begin
        if Customer.Name = '' then
            Error('Customer name is required');

        if Customer."E-Mail" = '' then
            Error('Customer email is required');
    end;
}
```

### Control Structure Indentation
```al
procedure ProcessOrderLines(OrderNo: Code[20])
var
    SalesLine: Record "Sales Line";
    Item: Record Item;
begin
    SalesLine.SetRange("Document Type", SalesLine."Document Type"::Order);
    SalesLine.SetRange("Document No.", OrderNo);

    if SalesLine.FindSet() then
        repeat
            if SalesLine.Type = SalesLine.Type::Item then begin
                if Item.Get(SalesLine."No.") then begin
                    if Item.Inventory < SalesLine.Quantity then begin
                        Error('Insufficient inventory for item %1', SalesLine."No.");
                    end else begin
                        Item.Inventory -= SalesLine.Quantity;
                        Item.Modify();
                    end;
                end;
            end;
        until SalesLine.Next() = 0;
end;
```

### Case Statement Indentation
```al
procedure HandleDocumentStatus(Status: Option Open,Released,Pending,Cancelled)
begin
    case Status of
        Status::Open:
            begin
                EnableEditingActions();
                ShowStatusMessage('Document is open for editing');
            end;
        Status::Released:
            begin
                DisableEditingActions();
                ShowStatusMessage('Document has been released');
            end;
        Status::Pending:
            begin
                DisableEditingActions();
                ShowStatusMessage('Document is pending approval');
                RequestApproval();
            end;
        Status::Cancelled:
            begin
                DisableAllActions();
                ShowStatusMessage('Document has been cancelled');
            end;
    end;
end;
```

## Bad Examples

### Inconsistent Trigger Indentation
```al
table 50101 "Bad Indentation Table"
{
    fields
    {
        field(1; "No."; Code[20]) { }
    }

trigger OnInsert()  // Wrong: should be indented
begin
if "No." = '' then  // Wrong: should be indented further
"No." := NoSeriesMgt.GetNextNo(Setup."Customer Nos.", 0D, true);  // Wrong: should be indented
end;

    trigger OnModify()  // Wrong: over-indented
        begin  // Wrong: over-indented
            "Last Modified Date" := Today;  // Inconsistent with other triggers
        end;
}
```

### Poor Procedure Indentation
```al
codeunit 50101 "Bad Indentation Codeunit"
{
procedure CreateCustomer(Name: Text[100]): Code[20]  // Wrong: not indented
var
Customer: Record Customer;  // Wrong: not indented
begin
Customer.Init();  // Wrong: not indented
    Customer.Name := Name;  // Inconsistent indentation
        Customer.Insert(true);  // Wrong: over-indented
exit(Customer."No.");  // Wrong: not indented
end;

    local procedure ValidateData()  // Wrong: over-indented
    begin
    // Implementation with poor indentation
    end;
}
```

### Inconsistent Control Structure Indentation
```al
procedure BadControlStructures(OrderNo: Code[20])
var
    SalesLine: Record "Sales Line";
begin
SalesLine.SetRange("Document No.", OrderNo);  // Wrong: not indented

if SalesLine.FindSet() then  // Wrong: not indented
repeat  // Wrong: not indented
if SalesLine.Type = SalesLine.Type::Item then  // Wrong: not indented
begin  // Wrong: not indented
        ProcessItem(SalesLine."No.");  // Wrong: over-indented
    end  // Wrong: inconsistent with begin
        else  // Wrong: poor else positioning
    begin  // Wrong: inconsistent indentation
        ProcessNonItem(SalesLine);
    end;
    until SalesLine.Next() = 0;  // Wrong: inconsistent with repeat
end;
```

### Poor Case Statement Indentation
```al
procedure BadCaseIndentation(Status: Option Open,Released,Pending)
begin
case Status of  // Wrong: not indented
Status::Open:  // Wrong: not indented further
begin  // Wrong: not indented to match case branch
Message('Open');  // Wrong: not indented properly in begin block
end;
    Status::Released:  // Wrong: inconsistent with other case labels
        begin  // Wrong: over-indented
            Message('Released');
        end;
Status::Pending:  // Wrong: back to no indentation
Message('Pending');  // Wrong: single statement not aligned
end;
end;
```

## Best Practices

1. **Consistent base indentation** for all keywords within their scope
2. **Progressive indentation** for nested structures (4 spaces per level)
3. **Align related keywords** (begin/end, if/then/else, case/of/end)
4. **Maintain indentation hierarchy** throughout complex structures
5. **Use consistent spacing** for all control flow keywords