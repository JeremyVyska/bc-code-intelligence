# TestField Error Handling Patterns

## Try-Catch with TestField
```al
procedure ValidateDocumentSafely(var SalesHeader: Record "Sales Header"): Boolean
begin
    if not TryValidateDocument(SalesHeader) then begin
        LogValidationError(GetLastErrorText());
        exit(false);
    end;
    exit(true);
end;

local procedure TryValidateDocument(var SalesHeader: Record "Sales Header")
begin
    SalesHeader.TestField("Document Type");
    SalesHeader.TestField("No.");
    SalesHeader.TestField("Sell-to Customer No.");
end;
```

## Batch Validation with Error Collection
```al
procedure ValidateCustomerBatch(var TempCustomer: Record Customer temporary): Text
var
    ErrorMessages: Text;
    Customer: Record Customer;
begin
    TempCustomer.Reset();
    if TempCustomer.FindSet() then
        repeat
            Customer := TempCustomer;
            if not TryValidateCustomer(Customer) then
                ErrorMessages += StrSubstNo('Customer %1: %2\', Customer."No.", GetLastErrorText());
        until TempCustomer.Next() = 0;
    
    exit(ErrorMessages);
end;

local procedure TryValidateCustomer(Customer: Record Customer): Boolean
begin
    Customer.TestField(Name);
    Customer.TestField("Customer Posting Group");
    exit(true);
end;
```

## Conditional TestField with Fallback
```al
procedure ValidateVendorWithDefaults(var Vendor: Record Vendor)
var
    VendorPostingGroup: Record "Vendor Posting Group";
begin
    if not TryTestVendorFields(Vendor) then begin
        // Apply defaults and retry
        if Vendor."Vendor Posting Group" = '' then begin
            VendorPostingGroup.FindFirst();
            Vendor."Vendor Posting Group" := VendorPostingGroup.Code;
        end;
        
        // Retry validation
        Vendor.TestField("Vendor Posting Group");
    end;
end;

local procedure TryTestVendorFields(Vendor: Record Vendor): Boolean
begin
    Vendor.TestField("Vendor Posting Group");
    exit(true);
end;
```

## Graceful Degradation Pattern
```al
procedure ProcessOrderWithValidation(var SalesHeader: Record "Sales Header"): Boolean
var
    ValidationLevel: Integer;
begin
    // Try strict validation first
    ValidationLevel := 1;
    if TryValidateOrderStrict(SalesHeader) then
        exit(true);
    
    // Fall back to basic validation
    ValidationLevel := 2;
    if TryValidateOrderBasic(SalesHeader) then begin
        LogWarning('Order %1 processed with basic validation only', SalesHeader."No.");
        exit(true);
    end;
    
    exit(false);
end;

local procedure TryValidateOrderStrict(SalesHeader: Record "Sales Header"): Boolean
begin
    SalesHeader.TestField("External Document No.");
    SalesHeader.TestField("Requested Delivery Date");
    SalesHeader.TestField("Shipment Method Code");
    exit(true);
end;

local procedure TryValidateOrderBasic(SalesHeader: Record "Sales Header"): Boolean
begin
    SalesHeader.TestField("Sell-to Customer No.");
    SalesHeader.TestField("Order Date");
    exit(true);
end;
```