# FieldError Default Messages - AL Code Sample

## Empty Field Default Messages

```al
table 50200 "Document Header"
{
    fields
    {
        field(1; "Document Type"; Enum "Sales Document Type") { }
        field(2; "Document No."; Code[20]) { }
        field(3; "Customer No."; Code[20]) { }
        field(10; "Posting Date"; Date) { }
        field(20; "Amount"; Decimal) { }
        field(30; "Currency Code"; Code[10]) { }
        field(40; "Description"; Text[100]) { }
    }
    
    keys
    {
        key(PK; "Document Type", "Document No.") { Clustered = true; }
    }
    
    procedure DemonstrateEmptyFieldMessages()
    begin
        // For empty Code field - generates specific "must have a value" message
        "Customer No." := '';
        // Output: "Customer No. must have a value in Document Header Document Type='Order', Document No.='DOC001'."
        FieldError("Customer No.");
        
        // For zero Date field - generates specific "must be filled in" message  
        "Posting Date" := 0D;
        // Output: "Posting Date must be filled in in Document Header Document Type='Order', Document No.='DOC001'."
        FieldError("Posting Date");
        
        // For zero Decimal field - generates "must not be 0" message
        Amount := 0;
        // Output: "Amount must not be 0 in Document Header Document Type='Order', Document No.='DOC001'."
        FieldError(Amount);
        
        // For empty Text field - generates "must have a value" message
        Description := '';
        // Output: "Description must have a value in Document Header Document Type='Order', Document No.='DOC001'."
        FieldError(Description);
    end;
}
```

## Non-Empty Field Default Messages

```al
table 50201 "Validation Examples"
{
    fields
    {
        field(1; "Entry No."; Integer) { }
        field(10; "Status"; Option) { OptionMembers = " ",Open,Released,Closed; }
        field(20; "Item No."; Code[20]) { }
        field(30; "Quantity"; Decimal) { }
        field(40; "Unit Price"; Decimal) { }
    }
    
    keys
    {
        key(PK; "Entry No.") { Clustered = true; }
    }
    
    procedure DemonstrateNonEmptyFieldMessages()
    begin
        "Entry No." := 1001;
        "Item No." := 'ITEM001';
        Quantity := 5.5;
        "Unit Price" := 100.00;
        Status := Status::Released;
        
        // For non-empty Code field - generates "is not valid" message
        // Output: "Item No. ITEM001 is not valid in Validation Examples Entry No.='1001'."
        FieldError("Item No.");
        
        // For non-zero Decimal field - generates "is not valid" message  
        // Output: "Quantity 5.5 is not valid in Validation Examples Entry No.='1001'."
        FieldError(Quantity);
        
        // For non-zero Option field - generates "is not valid" message
        // Output: "Status Released is not valid in Validation Examples Entry No.='1001'."
        FieldError(Status);
    end;
}
```

## Record Context in Error Messages

```al
table 50202 "Customer Transaction"
{
    fields
    {
        field(1; "Customer No."; Code[20]) { }
        field(2; "Transaction Date"; Date) { }
        field(3; "Transaction No."; Code[20]) { }
        field(10; "Amount"; Decimal) { }
        field(20; "Currency Code"; Code[10]) { }
    }
    
    keys
    {
        key(PK; "Customer No.", "Transaction Date", "Transaction No.") { Clustered = true; }
    }
    
    procedure DemonstrateRecordContextMessages()
    begin
        "Customer No." := 'CUST001';
        "Transaction Date" := Today;
        "Transaction No." := 'TXN001';
        Amount := 0;
        
        // BC automatically includes primary key fields in error message
        // Output: "Amount must not be 0 in Customer Transaction Customer No.='CUST001', Transaction Date='01/28/25', Transaction No.='TXN001'."
        FieldError(Amount);
        
        // Even with custom message, record context is automatically included
        // Output: "Amount cannot be zero for transactions in Customer Transaction Customer No.='CUST001', Transaction Date='01/28/25', Transaction No.='TXN001'."
        FieldError(Amount, 'cannot be zero for transactions');
    end;
}
```

## Default vs Custom Message Comparison

```al
codeunit 50200 "Message Comparison Examples"
{
    procedure CompareDefaultAndCustomMessages()
    var
        SalesHeader: Record "Sales Header";
        Customer: Record Customer;
    begin
        SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
        SalesHeader."No." := 'SO001';
        SalesHeader."Sell-to Customer No." := '';
        
        // DEFAULT MESSAGE EXAMPLES
        
        // Empty field default message
        // "Sell-to Customer No. must have a value in Sales Header Document Type='Order', No.='SO001'."
        // SalesHeader.FieldError("Sell-to Customer No.");
        
        // Non-empty field default message  
        SalesHeader."Sell-to Customer No." := 'INVALID';
        // "Sell-to Customer No. INVALID is not valid in Sales Header Document Type='Order', No.='SO001'."
        // SalesHeader.FieldError("Sell-to Customer No.");
        
        // CUSTOM MESSAGE EXAMPLES
        
        // Custom message with empty field
        SalesHeader."Sell-to Customer No." := '';
        // "Sell-to Customer No. is required for sales orders in Sales Header Document Type='Order', No.='SO001'."
        // SalesHeader.FieldError("Sell-to Customer No.", 'is required for sales orders');
        
        // Custom message with non-empty field
        SalesHeader."Sell-to Customer No." := 'BLOCKED';
        // "Sell-to Customer No. customer is blocked in Sales Header Document Type='Order', No.='SO001'."
        // SalesHeader.FieldError("Sell-to Customer No.", 'customer is blocked');
        
        // Complex custom message with context
        if not Customer.Get(SalesHeader."Sell-to Customer No.") then
            // "Sell-to Customer No. does not exist in customer master in Sales Header Document Type='Order', No.='SO001'."
            SalesHeader.FieldError("Sell-to Customer No.", 'does not exist in customer master');
    end;
}
```

## Real-World Message Generation Scenarios

```al
codeunit 50201 "Real-World Validation Messages"
{
    // Scenario 1: Document Processing Validation
    procedure ValidateDocumentForPosting(var SalesHeader: Record "Sales Header")
    begin
        // Required field validations - using default messages for consistency
        if SalesHeader."Sell-to Customer No." = '' then
            // "Sell-to Customer No. must have a value in Sales Header..."
            SalesHeader.FieldError("Sell-to Customer No.");
            
        if SalesHeader."Posting Date" = 0D then  
            // "Posting Date must be filled in in Sales Header..."
            SalesHeader.FieldError("Posting Date");
            
        // Business rule validations - using custom messages for clarity
        if SalesHeader."Posting Date" > WorkDate() + 30 then
            // "Posting Date cannot be more than 30 days in future in Sales Header..."
            SalesHeader.FieldError("Posting Date", 'cannot be more than 30 days in future');
    end;
    
    // Scenario 2: Setup Validation with Detailed Messages
    procedure ValidateCustomerSetup(var Customer: Record Customer)
    var
        PostCode: Record "Post Code";
    begin
        // Default message for required field
        if Customer."No." = '' then
            // "No. must have a value in Customer..."
            Customer.FieldError("No.");
            
        // Custom message for business logic
        if Customer."Credit Limit (LCY)" < 0 then
            // "Credit Limit (LCY) cannot be negative in Customer No.='CUST001'."
            Customer.FieldError("Credit Limit (LCY)", 'cannot be negative');
            
        // Validation with lookup and detailed custom message
        if (Customer."Post Code" <> '') and not PostCode.Get(Customer."Post Code", Customer.City) then
            // "Post Code is not valid for the specified city in Customer No.='CUST001'."
            Customer.FieldError("Post Code", 'is not valid for the specified city');
    end;
    
    // Scenario 3: Line Validation with Context
    procedure ValidateSalesLine(var SalesLine: Record "Sales Line")
    var
        Item: Record Item;
        AvailableInventory: Decimal;
    begin
        // Standard required field validation
        if SalesLine."No." = '' then
            // "No. must have a value in Sales Line Document Type='Order', Document No.='SO001', Line No.='10000'."
            SalesLine.FieldError("No.");
            
        // Business logic with calculated values in message
        if SalesLine.Type = SalesLine.Type::Item then begin
            Item.Get(SalesLine."No.");
            Item.CalcFields(Inventory);
            AvailableInventory := Item.Inventory;
            
            if SalesLine.Quantity > AvailableInventory then
                // "Quantity exceeds available inventory (50 units available) in Sales Line..."
                SalesLine.FieldError(Quantity, 
                    StrSubstNo('exceeds available inventory (%1 units available)', AvailableInventory));
        end;
        
        // Price validation with business context
        if SalesLine."Unit Price" <= 0 then
            // "Unit Price must be positive for sales transactions in Sales Line..."
            SalesLine.FieldError("Unit Price", 'must be positive for sales transactions');
    end;
}
```

## Message Formatting Rules and Patterns

```al
codeunit 50202 "Message Formatting Examples"
{
    procedure DemonstrateFormattingRules()
    var
        Item: Record Item;
        PurchaseLine: Record "Purchase Line";
    begin
        Item."No." := 'ITEM001';
        
        // RULE 1: Default messages automatically include field value for non-empty fields
        Item.Description := 'Invalid Description Content';
        // "Description Invalid Description Content is not valid in Item No.='ITEM001'."
        // Item.FieldError(Description);
        
        // RULE 2: Custom messages start with lowercase (BC convention)
        // CORRECT:
        // "Description must not contain special characters in Item No.='ITEM001'."
        // Item.FieldError(Description, 'must not contain special characters');
        
        // INCORRECT (would still work but doesn't follow BC conventions):
        // Item.FieldError(Description, 'Must not contain special characters');
        
        // RULE 3: Record context automatically appended using primary key
        PurchaseLine."Document Type" := PurchaseLine."Document Type"::Order;
        PurchaseLine."Document No." := 'PO001';
        PurchaseLine."Line No." := 10000;
        PurchaseLine.Quantity := -5;
        
        // "Quantity cannot be negative in Purchase Line Document Type='Order', Document No.='PO001', Line No.='10000'."
        PurchaseLine.FieldError(Quantity, 'cannot be negative');
        
        // RULE 4: Multiple primary key fields included in order
        // All key fields shown in error context for complete record identification
    end;
    
    procedure DemonstrateMessageCustomization()
    var
        Customer: Record Customer;
        CreditLimit: Decimal;
    begin
        Customer."No." := 'CUST001';
        Customer."Credit Limit (LCY)" := 5000;
        CreditLimit := 10000;  // Required limit for this customer
        
        // Pattern: Specific business context in custom messages
        // "Credit Limit (LCY) insufficient for customer risk category (minimum 10000 required) in Customer No.='CUST001'."
        Customer.FieldError("Credit Limit (LCY)", 
            StrSubstNo('insufficient for customer risk category (minimum %1 required)', CreditLimit));
        
        // Pattern: Action guidance in custom messages
        // "Credit Limit (LCY) requires manager approval for amounts above 50000 in Customer No.='CUST001'."
        Customer.FieldError("Credit Limit (LCY)", 'requires manager approval for amounts above 50000');
        
        // Pattern: Reference to related data in custom messages  
        // "Credit Limit (LCY) conflicts with existing payment terms (Net 60) in Customer No.='CUST001'."
        Customer.FieldError("Credit Limit (LCY)", 'conflicts with existing payment terms (Net 60)');
    end;
}
```

## Advanced Default Message Scenarios

```al
table 50203 "Advanced Message Examples"
{
    fields
    {
        field(1; "Primary Key"; Code[20]) { }
        field(10; "Boolean Field"; Boolean) { }
        field(20; "DateTime Field"; DateTime) { }
        field(30; "GUID Field"; Guid) { }
        field(40; "Blob Field"; Blob) { }
    }
    
    procedure DemonstrateAdvancedDefaultMessages()
    begin
        "Primary Key" := 'TEST001';
        
        // Boolean field default messages
        "Boolean Field" := false;
        // "Boolean Field No is not valid in Advanced Message Examples Primary Key='TEST001'."
        // FieldError("Boolean Field");
        
        "Boolean Field" := true;  
        // "Boolean Field Yes is not valid in Advanced Message Examples Primary Key='TEST001'."
        // FieldError("Boolean Field");
        
        // DateTime field default messages
        "DateTime Field" := 0DT;
        // "DateTime Field must be filled in in Advanced Message Examples Primary Key='TEST001'."
        // FieldError("DateTime Field");
        
        "DateTime Field" := CurrentDateTime;
        // "DateTime Field 01/28/25 09:30:00 is not valid in Advanced Message Examples Primary Key='TEST001'."
        // FieldError("DateTime Field");
        
        // GUID field default messages
        Clear("GUID Field");
        // "GUID Field must have a value in Advanced Message Examples Primary Key='TEST001'."
        // FieldError("GUID Field");
        
        "GUID Field" := CreateGuid();
        // "GUID Field {12345678-1234-1234-1234-123456789012} is not valid in Advanced Message Examples Primary Key='TEST001'."
        // FieldError("GUID Field");
    end;
}