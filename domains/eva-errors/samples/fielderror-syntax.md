# FieldError Method Syntax - AL Code Sample

## Basic FieldError Syntax

```al
// Basic FieldError syntax with automatic message
table 50100 "Sales Document"
{
    fields
    {
        field(1; "Document Type"; Enum "Sales Document Type") { }
        field(2; "Document No."; Code[20]) { }
        field(3; "Sell-to Customer No."; Code[20]) { }
        field(10; "Currency Code"; Code[10]) { }
        field(20; "Amount"; Decimal) { }
    }
    
    trigger OnInsert()
    begin
        // Validate required fields before insert
        ValidateRequiredFields();
    end;
    
    local procedure ValidateRequiredFields()
    begin
        // FieldError assumes validation already failed - no testing performed
        if "Sell-to Customer No." = '' then
            FieldError("Sell-to Customer No.");  // Generates: "Sell-to Customer No. must have a value in Sales Document Document Type='Order', Document No.='SO001'."
            
        if Amount <= 0 then
            FieldError(Amount);  // Generates: "Amount must not be 0 in Sales Document..."
    end;
}
```

## FieldError vs TestField Comparison

```al
codeunit 50100 "Validation Examples"
{
    procedure DemonstrateFieldErrorVsTestField()
    var
        SalesHeader: Record "Sales Header";
        Customer: Record Customer;
    begin
        SalesHeader.Get(SalesHeader."Document Type"::Order, 'SO001');
        
        // TestField approach - tests condition AND throws error if failed
        SalesHeader.TestField("Sell-to Customer No.");  // Tests if empty, then errors
        SalesHeader.TestField(Amount);  // Tests if zero, then errors
        
        // FieldError approach - assumes you already validated the condition
        if SalesHeader."Sell-to Customer No." = '' then
            SalesHeader.FieldError("Sell-to Customer No.");  // Only errors, no testing
            
        if SalesHeader.Amount <= 0 then
            SalesHeader.FieldError(Amount, 'must be positive');  // Custom message
            
        // Practical usage: FieldError for complex validation logic
        if not Customer.Get(SalesHeader."Sell-to Customer No.") then
            SalesHeader.FieldError("Sell-to Customer No.", 'customer does not exist');
            
        if Customer.Blocked <> Customer.Blocked::" " then
            SalesHeader.FieldError("Sell-to Customer No.", 
                StrSubstNo('customer is blocked (%1)', Customer.Blocked));
    end;
}
```

## Custom Message Examples

```al
table 50101 "Purchase Document"
{
    fields
    {
        field(1; "Document Type"; Enum "Purchase Document Type") { }
        field(2; "Document No."; Code[20]) { }
        field(3; "Buy-from Vendor No."; Code[20]) { }
        field(10; "Expected Receipt Date"; Date) { }
        field(20; "Amount Including VAT"; Decimal) { }
    }
    
    procedure ValidateBusinessRules()
    var
        Vendor: Record Vendor;
    begin
        // Custom message examples - note lowercase first word convention
        if "Expected Receipt Date" < WorkDate() then
            FieldError("Expected Receipt Date", 'cannot be in the past');
            
        if "Amount Including VAT" > 10000 then
            FieldError("Amount Including VAT", 'exceeds approval limit');
            
        // Complex validation with detailed custom message
        if Vendor.Get("Buy-from Vendor No.") then begin
            if Vendor.Blocked <> Vendor.Blocked::" " then
                FieldError("Buy-from Vendor No.", 
                    StrSubstNo('vendor is blocked for %1', Vendor.Blocked));
                    
            if (Vendor."Payment Terms Code" = '') and ("Amount Including VAT" > 1000) then
                FieldError("Buy-from Vendor No.", 
                    'requires payment terms for amounts over 1000');
        end else
            FieldError("Buy-from Vendor No.", 'vendor does not exist');
    end;
}
```

## Real-World Posting Codeunit Pattern

```al
codeunit 50102 "Sales Document Validation"
{
    // Pattern used in standard BC posting codeunits (12, 80, 90)
    
    procedure ValidateSalesDocument(var SalesHeader: Record "Sales Header")
    begin
        // Comprehensive validation before posting
        with SalesHeader do begin
            // Required field validation
            if "Sell-to Customer No." = '' then
                FieldError("Sell-to Customer No.");
                
            if "Document Date" = 0D then
                FieldError("Document Date");
                
            // Business logic validation with custom messages
            if "Posting Date" < "Document Date" then
                FieldError("Posting Date", 'cannot be before document date');
                
            if Status <> Status::Released then
                FieldError(Status, 'must be Released before posting');
                
            // Complex validation scenarios
            ValidateCustomerCredit(SalesHeader);
            ValidateDimensions(SalesHeader);
        end;
    end;
    
    local procedure ValidateCustomerCredit(var SalesHeader: Record "Sales Header")
    var
        Customer: Record Customer;
        CustLedgerEntry: Record "Cust. Ledger Entry";
        RemainingAmount: Decimal;
    begin
        Customer.Get(SalesHeader."Sell-to Customer No.");
        
        // Calculate customer's remaining credit
        CustLedgerEntry.SetRange("Customer No.", Customer."No.");
        CustLedgerEntry.SetRange(Open, true);
        CustLedgerEntry.CalcSums("Remaining Amt. (LCY)");
        RemainingAmount := CustLedgerEntry."Remaining Amt. (LCY)";
        
        // Validate credit limit
        if (Customer."Credit Limit (LCY)" > 0) and 
           (RemainingAmount + SalesHeader."Amount Including VAT" > Customer."Credit Limit (LCY)") then
            SalesHeader.FieldError("Sell-to Customer No.", 
                StrSubstNo('credit limit exceeded. Available credit: %1', 
                          Customer."Credit Limit (LCY)" - RemainingAmount));
    end;
    
    local procedure ValidateDimensions(var SalesHeader: Record "Sales Header")
    var
        DimMgt: Codeunit DimensionManagement;
        DimSetID: Integer;
    begin
        DimSetID := SalesHeader."Dimension Set ID";
        
        // Validate required dimensions
        if not DimMgt.CheckDimIDComb(DimSetID) then
            SalesHeader.FieldError("Dimension Set ID", 'invalid dimension combination');
            
        // Validate dimension values against posting rules
        if not DimMgt.CheckDimValuePosting(DimSetID, DATABASE::"Sales Header", SalesHeader."No.") then
            SalesHeader.FieldError("Dimension Set ID", 'dimension values not allowed for posting');
    end;
}
```

## Advanced FieldError Patterns

```al
codeunit 50103 "Advanced Validation Patterns"
{
    // Pattern: Conditional validation with detailed context
    procedure ValidateItemAvailability(var SalesLine: Record "Sales Line")
    var
        Item: Record Item;
        AvailableQty: Decimal;
    begin
        if SalesLine.Type <> SalesLine.Type::Item then
            exit;  // Only validate items
            
        Item.Get(SalesLine."No.");
        Item.CalcFields(Inventory, "Reserved Qty. on Inventory");
        AvailableQty := Item.Inventory - Item."Reserved Qty. on Inventory";
        
        if SalesLine.Quantity > AvailableQty then
            SalesLine.FieldError(Quantity, 
                StrSubstNo('exceeds available inventory (%1 available)', AvailableQty));
    end;
    
    // Pattern: Multi-field validation with specific field error
    procedure ValidatePriceAndDiscount(var SalesLine: Record "Sales Line")
    begin
        // Validate that discount doesn't exceed unit price
        if (SalesLine."Unit Price" > 0) and 
           (SalesLine."Line Discount Amount" > SalesLine."Unit Price" * SalesLine.Quantity) then
            SalesLine.FieldError("Line Discount Amount", 
                'cannot exceed line amount');
                
        // Validate price against minimum price rules
        if SalesLine."Unit Price" < GetMinimumPrice(SalesLine) then
            SalesLine.FieldError("Unit Price", 
                StrSubstNo('cannot be below minimum price (%1)', GetMinimumPrice(SalesLine)));
    end;
    
    local procedure GetMinimumPrice(SalesLine: Record "Sales Line"): Decimal
    var
        Item: Record Item;
    begin
        if Item.Get(SalesLine."No.") then
            exit(Item."Unit Cost" * 1.1)  // 10% markup minimum
        else
            exit(0);
    end;
}
```

## Error Context and Message Output Examples

```al
// When this validation fails:
procedure ExampleValidationScenarios()
var
    SalesHeader: Record "Sales Header";
begin
    SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
    SalesHeader."No." := 'SO001';
    SalesHeader."Sell-to Customer No." := '';  // Empty value
    
    // This will generate message:
    // "Sell-to Customer No. must have a value in Sales Header Document Type='Order', No.='SO001'."
    SalesHeader.FieldError("Sell-to Customer No.");
    
    // With custom message:
    // "Sell-to Customer No. is required for orders in Sales Header Document Type='Order', No.='SO001'."
    SalesHeader.FieldError("Sell-to Customer No.", 'is required for orders');
    
    // For non-empty field with custom message:
    SalesHeader."Sell-to Customer No." := 'INVALID';
    // "Sell-to Customer No. customer does not exist in Sales Header Document Type='Order', No.='SO001'."
    SalesHeader.FieldError("Sell-to Customer No.", 'customer does not exist');
end;
```