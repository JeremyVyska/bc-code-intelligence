# FieldError Syntax Usage Examples

## Basic FieldError Usage
```al
trigger OnValidate()
begin
    if Quantity <= 0 then
        FieldError(Quantity, 'Quantity must be greater than zero.');
end;
```

## FieldError with Business Context
```al
trigger OnValidate()
begin
    if ("Starting Date" > "Ending Date") and ("Ending Date" <> 0D) then
        FieldError("Starting Date", 'Starting date cannot be later than ending date.');
end;
```

## FieldError in Complex Validation
```al
procedure ValidateInventoryPosting()
var
    Location: Record Location;
begin
    if "Location Code" <> '' then begin
        if not Location.Get("Location Code") then
            FieldError("Location Code", 'Location does not exist.');
        
        if not Location."Require Shipment" and ("Bin Code" = '') then
            FieldError("Bin Code", 'Bin code is required for this location.');
    end;
end;
```

## FieldError with User Guidance
```al
trigger OnValidate()
begin
    Customer.SetRange("No.", "Customer No.");
    if not Customer.FindFirst() then
        FieldError("Customer No.", 'Customer not found. Please select a valid customer from the lookup.');
        
    if Customer.Blocked <> Customer.Blocked::" " then
        FieldError("Customer No.", StrSubstNo('Customer %1 is blocked for %2.', Customer."No.", Customer.Blocked));
end;
```