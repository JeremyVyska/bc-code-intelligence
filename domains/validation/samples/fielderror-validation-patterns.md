# FieldError Validation Patterns

## OnValidate Trigger Integration
```al
trigger OnValidate()
begin
    if "Starting Date" > "Ending Date" then
        FieldError("Starting Date", 'Starting date must be earlier than or equal to ending date.');
        
    if ("Starting Date" < WorkDate()) and ("Document Status" = "Document Status"::Draft) then
        FieldError("Starting Date", 'Starting date cannot be in the past for draft documents.');
end;
```

## Cross-Field Validation
```al
trigger OnValidate()
var
    Item: Record Item;
begin
    if "Item No." <> '' then begin
        Item.Get("Item No.");
        
        if (Item.Type = Item.Type::Service) and ("Location Code" <> '') then
            FieldError("Location Code", 'Location cannot be specified for service items.');
            
        if (Item."Item Tracking Code" <> '') and ("Lot No." = '') and ("Serial No." = '') then
            FieldError("Lot No.", 'Either lot number or serial number must be specified for tracked items.');
    end;
end;
```

## Business Rule Enforcement
```al
procedure ValidateCustomerCreditLimit()
var
    Customer: Record Customer;
    CustomerLedgerEntry: Record "Cust. Ledger Entry";
    OutstandingAmount: Decimal;
begin
    Customer.Get("Customer No.");
    
    if Customer."Credit Limit (LCY)" = 0 then
        exit; // No credit limit enforced
        
    CustomerLedgerEntry.SetRange("Customer No.", "Customer No.");
    CustomerLedgerEntry.SetRange(Open, true);
    CustomerLedgerEntry.CalcSums("Remaining Amt. (LCY)");
    OutstandingAmount := CustomerLedgerEntry."Remaining Amt. (LCY)";
    
    if OutstandingAmount + "Amount Including VAT" > Customer."Credit Limit (LCY)" then
        FieldError("Amount Including VAT", 
            StrSubstNo('Transaction exceeds credit limit. Outstanding: %1, Credit Limit: %2, This Transaction: %3',
                OutstandingAmount, Customer."Credit Limit (LCY)", "Amount Including VAT"));
end;
```

## Posting Procedure Validation
```al
procedure ValidateForPosting()
begin
    TestField("Document Type");
    TestField("No.");
    
    case "Document Type" of
        "Document Type"::Quote:
            FieldError("Document Type", 'Quotes cannot be posted. Convert to order or invoice first.');
        "Document Type"::Order:
            ValidateOrderForPosting();
        "Document Type"::Invoice:
            ValidateInvoiceForPosting();
    end;
end;

local procedure ValidateOrderForPosting()
begin
    if "Shipment Date" = 0D then
        FieldError("Shipment Date", 'Shipment date is required for posting orders.');
        
    if not LinesExist() then
        FieldError("Document Type", 'Document must have at least one line to post.');
end;
```