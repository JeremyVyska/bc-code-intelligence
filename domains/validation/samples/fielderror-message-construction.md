# FieldError Message Construction Examples

## Descriptive Error Messages
```al
trigger OnValidate()
begin
    if "Unit Price" < 0 then
        FieldError("Unit Price", 'Unit price cannot be negative. Please enter a positive value.');
        
    if ("Discount %" < 0) or ("Discount %" > 100) then
        FieldError("Discount %", 'Discount percentage must be between 0 and 100.');
end;
```

## Context-Aware Messages
```al
procedure ValidateItemForLocation()
begin
    Item.Get("Item No.");
    Location.Get("Location Code");
    
    if not Location."Allow Negative Inventory" and (CalculateAvailableQty() < Quantity) then
        FieldError(Quantity, StrSubstNo('Insufficient inventory at location %1. Available: %2, Required: %3', 
            "Location Code", CalculateAvailableQty(), Quantity));
end;
```

## Business Rule Explanations
```al
trigger OnValidate()
var
    SalesSetup: Record "Sales & Receivables Setup";
begin
    SalesSetup.Get();
    
    if SalesSetup."Credit Warnings" = SalesSetup."Credit Warnings"::"No Warning" then
        exit;
        
    Customer.Get("Sell-to Customer No.");
    Customer.CalcFields("Balance (LCY)");
    
    if Customer."Balance (LCY)" > Customer."Credit Limit (LCY)" then
        FieldError("Sell-to Customer No.", 
            StrSubstNo('Customer %1 exceeds credit limit. Current balance: %2, Credit limit: %3. Please contact credit department.',
                Customer.Name, Customer."Balance (LCY)", Customer."Credit Limit (LCY)"));
end;
```

## Corrective Action Guidance
```al
procedure ValidateShippingSetup()
begin
    if "Shipment Method Code" = '' then
        FieldError("Shipment Method Code", 'Shipment method is required. Select a method from the dropdown or create a new one in Shipment Methods setup.');
        
    if ("Shipping Agent Code" <> '') and ("Shipping Agent Service Code" = '') then
        FieldError("Shipping Agent Service Code", 
            'Service code is required when shipping agent is specified. Use the lookup to select an available service.');
end;
```