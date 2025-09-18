# AL Blank Line Organization - Code Examples

## Good Examples

### Logical Section Separation
```al
codeunit 50100 "Customer Management"
{
    procedure CreateCustomer(Name: Text[100]; Email: Text[80]): Code[20]
    var
        Customer: Record Customer;
        CustomerNo: Code[20];
    begin
        // Input validation section
        if Name = '' then
            Error('Customer name cannot be empty');
        if Email = '' then
            Error('Customer email cannot be empty');

        // Customer creation section
        Customer.Init();
        CustomerNo := NoSeriesMgt.GetNextNo(SalesSetup."Customer Nos.", 0D, true);
        Customer."No." := CustomerNo;
        Customer.Name := Name;
        Customer."E-Mail" := Email;

        // Database operation section
        Customer.Insert(true);

        // Return result
        exit(CustomerNo);
    end;
}
```

### Function Group Separation
```al
codeunit 50101 "Order Processing"
{
    // Validation functions
    procedure ValidateOrderHeader(var SalesHeader: Record "Sales Header")
    begin
        if SalesHeader."Sell-to Customer No." = '' then
            Error('Customer number is required');
    end;

    procedure ValidateOrderLines(var SalesLine: Record "Sales Line")
    begin
        if SalesLine.Quantity <= 0 then
            Error('Quantity must be greater than zero');
    end;

    // Processing functions
    procedure ProcessOrder(var SalesHeader: Record "Sales Header")
    begin
        ValidateOrderHeader(SalesHeader);
        CalculateTotals(SalesHeader);
        UpdateInventory(SalesHeader);
    end;

    procedure CalculateTotals(var SalesHeader: Record "Sales Header")
    begin
        SalesHeader.CalcFields("Amount Including VAT");
    end;
}
```

## Bad Examples

### No Logical Separation
```al
codeunit 50102 "Bad Organization"
{
    procedure ProcessData()
    var
        Customer: Record Customer;
        Item: Record Item;
        SalesLine: Record "Sales Line";
    begin
        if Customer.FindFirst() then
            Customer.Modify();
        Item.SetRange(Blocked, false);
        if Item.FindSet() then
            repeat
                Item."Unit Price" := Item."Unit Price" * 1.1;
                Item.Modify();
            until Item.Next() = 0;
        SalesLine.SetRange("Document Type", SalesLine."Document Type"::Order);
        SalesLine.SetRange("Shipment Date", Today);
        if SalesLine.FindSet() then
            SalesLine.DeleteAll();
        Message('Processing complete');
    end;
}
```

### Excessive Blank Lines
```al
codeunit 50103 "Excessive Spacing"
{
    procedure BadSpacing()
    var
        Customer: Record Customer;


        Item: Record Item;


        Result: Boolean;
    begin


        Customer.FindFirst();


        Item.FindFirst();


        Result := true;


    end;


}
```

### Missing Function Separation
```al
codeunit 50104 "No Function Separation"
{
    procedure ValidateCustomer(CustomerNo: Code[20]): Boolean
    begin
        exit(CustomerNo <> '');
    end;
    procedure CreateItem(ItemNo: Code[20]; Description: Text[100])
    var
        Item: Record Item;
    begin
        Item.Init();
        Item."No." := ItemNo;
        Item.Description := Description;
        Item.Insert();
    end;
    procedure DeleteOrder(OrderNo: Code[20])
    var
        SalesHeader: Record "Sales Header";
    begin
        SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo);
        SalesHeader.Delete(true);
    end;
}
```

## Best Practices

1. **Separate logical sections** within procedures with single blank lines
2. **Separate functions** with single blank lines between procedure declarations
3. **Group related functions** together without excessive spacing
4. **Use consistent spacing** throughout the codeunit
5. **Avoid excessive blank lines** that disrupt reading flow