# TestField Basic Syntax Examples

## Simple Field Validation
```al
procedure ValidateCustomerData()
var
    Customer: Record Customer;
begin
    Customer.TestField("No.");
    Customer.TestField(Name);
    Customer.TestField("Customer Posting Group");
end;
```

## Field Value Validation
```al
procedure ValidateItemType()
var
    Item: Record Item;
begin
    Item.TestField(Type, Item.Type::Inventory);
    Item.TestField("Inventory Posting Group");
end;
```

## Multiple Field Validation
```al
procedure ValidateVendorSetup()
var
    Vendor: Record Vendor;
begin
    Vendor.TestField("No.");
    Vendor.TestField(Name);
    Vendor.TestField("Vendor Posting Group");
    Vendor.TestField("Payment Terms Code");
end;
```

## Conditional Field Validation
```al
procedure ValidateCustomerBalance()
var
    Customer: Record Customer;
begin
    Customer.CalcFields(Balance);
    if Customer."Credit Limit (LCY)" > 0 then
        Customer.TestField(Balance);
end;
```