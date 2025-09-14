# API Page Validation Rule Execution - AL Code Sample

## Basic Validation Rule Implementation

```al
page 50500 "Validated Customer API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'customers';
    APIVersion = 'v1.0';
    EntityName = 'customer';
    EntitySetName = 'customers';
    SourceTable = Customer;
    DelayedInsert = true;
    ODataKeyFields = SystemId;
    
    layout
    {
        area(Content)
        {
            repeater(GroupName)
            {
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                    Editable = false;
                }
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                    // Field validation automatically triggered by BC
                }
                field(displayName; Rec.Name)
                {
                    Caption = 'Display Name';
                    // Required field validation
                }
                field(email; Rec."E-Mail")
                {
                    Caption = 'Email';
                    // Email format validation through table field definition
                }
                field(creditLimit; Rec."Credit Limit (LCY)")
                {
                    Caption = 'Credit Limit';
                    // Numeric validation and business rules
                }
                field(customerPostingGroup; Rec."Customer Posting Group")
                {
                    Caption = 'Customer Posting Group';
                    // TableRelation validation
                }
            }
        }
    }
    
    // Business validation rules executed during API operations
    trigger OnInsertRecord(BelowxRec: Boolean): Boolean
    var
        CustomerValidation: Codeunit "Customer Validation Rules";
    begin
        // Execute comprehensive business validation
        if not CustomerValidation.ValidateNewCustomer(Rec) then
            exit(false);
            
        // Additional API-specific validations
        ValidateAPISpecificRules();
        
        exit(true);
    end;
    
    trigger OnModifyRecord(): Boolean
    var
        CustomerValidation: Codeunit "Customer Validation Rules";
    begin
        // Execute modification validation rules
        if not CustomerValidation.ValidateCustomerModification(Rec, xRec) then
            exit(false);
            
        ValidateAPISpecificRules();
        
        exit(true);
    end;
    
    trigger OnDeleteRecord(): Boolean
    var
        CustomerValidation: Codeunit "Customer Validation Rules";
    begin
        // Validate deletion is allowed
        if not CustomerValidation.ValidateCustomerDeletion(Rec) then
            exit(false);
            
        exit(true);
    end;
    
    local procedure ValidateAPISpecificRules()
    begin
        // API-specific validation rules
        if Rec.Name = '' then
            Error('Display name is required for API operations');
            
        if Rec."E-Mail" <> '' then
            if not IsValidEmail(Rec."E-Mail") then
                Error('Invalid email format: %1', Rec."E-Mail");
                
        // Credit limit validation
        if Rec."Credit Limit (LCY)" < 0 then
            Error('Credit limit cannot be negative');
    end;
    
    local procedure IsValidEmail(Email: Text): Boolean
    var
        EmailPattern: Text;
    begin
        EmailPattern := '*@*.*';
        exit(Email.Contains('@') and (StrLen(Email) > 5));
    end;
}

// Comprehensive validation rules codeunit
codeunit 50500 "Customer Validation Rules"
{
    // Validate new customer creation
    procedure ValidateNewCustomer(var Customer: Record Customer): Boolean
    begin
        // Required field validations
        if not ValidateRequiredFields(Customer) then
            exit(false);
            
        // Business rule validations
        if not ValidateBusinessRules(Customer) then
            exit(false);
            
        // Data integrity validations
        if not ValidateDataIntegrity(Customer) then
            exit(false);
            
        exit(true);
    end;
    
    // Validate customer modifications
    procedure ValidateCustomerModification(var Customer: Record Customer; OldCustomer: Record Customer): Boolean
    begin
        // Check if critical fields are being modified
        if Customer."No." <> OldCustomer."No." then
            Error('Customer number cannot be modified');
            
        // Validate changes don't violate business rules
        if Customer."Customer Posting Group" <> OldCustomer."Customer Posting Group" then
            if not ValidatePostingGroupChange(Customer, OldCustomer) then
                exit(false);
                
        // Validate credit limit changes
        if Customer."Credit Limit (LCY)" <> OldCustomer."Credit Limit (LCY)" then
            if not ValidateCreditLimitChange(Customer, OldCustomer) then
                exit(false);
                
        exit(ValidateNewCustomer(Customer));
    end;
    
    // Validate customer deletion
    procedure ValidateCustomerDeletion(var Customer: Record Customer): Boolean
    var
        SalesLine: Record "Sales Line";
        CustLedgerEntry: Record "Cust. Ledger Entry";
    begin
        // Check for existing transactions
        CustLedgerEntry.SetRange("Customer No.", Customer."No.");
        if not CustLedgerEntry.IsEmpty() then
            Error('Cannot delete customer %1. Customer has ledger entries.', Customer."No.");
            
        // Check for open sales documents
        SalesLine.SetRange("Sell-to Customer No.", Customer."No.");
        if not SalesLine.IsEmpty() then
            Error('Cannot delete customer %1. Customer has open sales documents.', Customer."No.");
            
        exit(true);
    end;
    
    local procedure ValidateRequiredFields(var Customer: Record Customer): Boolean
    begin
        if Customer."No." = '' then
            Error('Customer number is required');
            
        if Customer.Name = '' then
            Error('Customer name is required');
            
        if Customer."Customer Posting Group" = '' then
            Error('Customer posting group is required');
            
        exit(true);
    end;
    
    local procedure ValidateBusinessRules(var Customer: Record Customer): Boolean
    var
        CustomerPostingGroup: Record "Customer Posting Group";
        PaymentTerms: Record "Payment Terms";
    begin
        // Validate posting group exists
        if not CustomerPostingGroup.Get(Customer."Customer Posting Group") then
            Error('Customer posting group %1 does not exist', Customer."Customer Posting Group");
            
        // Validate payment terms if specified
        if Customer."Payment Terms Code" <> '' then
            if not PaymentTerms.Get(Customer."Payment Terms Code") then
                Error('Payment terms %1 do not exist', Customer."Payment Terms Code");
                
        // Validate credit limit is reasonable
        if Customer."Credit Limit (LCY)" > 1000000 then
            Error('Credit limit exceeds maximum allowed amount of 1,000,000');
            
        exit(true);
    end;
    
    local procedure ValidateDataIntegrity(var Customer: Record Customer): Boolean
    var
        ExistingCustomer: Record Customer;
    begin
        // Check for duplicate customer numbers
        ExistingCustomer.SetRange("No.", Customer."No.");
        ExistingCustomer.SetFilter(SystemId, '<>%1', Customer.SystemId);
        if not ExistingCustomer.IsEmpty() then
            Error('Customer number %1 already exists', Customer."No.");
            
        // Validate email uniqueness if configured
        if Customer."E-Mail" <> '' then
            if not ValidateEmailUniqueness(Customer) then
                exit(false);
                
        exit(true);
    end;
    
    local procedure ValidatePostingGroupChange(var Customer: Record Customer; OldCustomer: Record Customer): Boolean
    var
        CustLedgerEntry: Record "Cust. Ledger Entry";
    begin
        // Prevent posting group change if customer has open entries
        CustLedgerEntry.SetRange("Customer No.", Customer."No.");
        CustLedgerEntry.SetRange(Open, true);
        if not CustLedgerEntry.IsEmpty() then
            Error('Cannot change posting group for customer %1. Customer has open ledger entries.', Customer."No.");
            
        exit(true);
    end;
    
    local procedure ValidateCreditLimitChange(var Customer: Record Customer; OldCustomer: Record Customer): Boolean
    var
        SalesHeader: Record "Sales Header";
        OutstandingAmount: Decimal;
    begin
        // If credit limit is being reduced, check outstanding orders
        if Customer."Credit Limit (LCY)" < OldCustomer."Credit Limit (LCY)" then begin
            SalesHeader.SetRange("Sell-to Customer No.", Customer."No.");
            SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
            if SalesHeader.FindSet() then
                repeat
                    SalesHeader.CalcFields("Amount Including VAT");
                    OutstandingAmount += SalesHeader."Amount Including VAT";
                until SalesHeader.Next() = 0;
                
            if OutstandingAmount > Customer."Credit Limit (LCY)" then
                Error('Cannot reduce credit limit below outstanding order amount of %1', OutstandingAmount);
        end;
        
        exit(true);
    end;
    
    local procedure ValidateEmailUniqueness(var Customer: Record Customer): Boolean
    var
        ExistingCustomer: Record Customer;
    begin
        ExistingCustomer.SetRange("E-Mail", Customer."E-Mail");
        ExistingCustomer.SetFilter(SystemId, '<>%1', Customer.SystemId);
        if not ExistingCustomer.IsEmpty() then
            Error('Email address %1 is already assigned to another customer', Customer."E-Mail");
            
        exit(true);
    end;
}
```

## Advanced Field-Level Validation

```al
page 50501 "Item Advanced Validation API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'inventory';
    APIVersion = 'v1.0';
    EntityName = 'item';
    EntitySetName = 'items';
    SourceTable = Item;
    DelayedInsert = true;
    ODataKeyFields = SystemId;
    
    layout
    {
        area(Content)
        {
            repeater(GroupName)
            {
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                    Editable = false;
                }
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                    
                    trigger OnValidate()
                    begin
                        ValidateItemNumber();
                    end;
                }
                field(description; Rec.Description)
                {
                    Caption = 'Description';
                    
                    trigger OnValidate()
                    begin
                        ValidateDescription();
                    end;
                }
                field(unitPrice; Rec."Unit Price")
                {
                    Caption = 'Unit Price';
                    
                    trigger OnValidate()
                    begin
                        ValidateUnitPrice();
                    end;
                }
                field(unitCost; Rec."Unit Cost")
                {
                    Caption = 'Unit Cost';
                    
                    trigger OnValidate()
                    begin
                        ValidateUnitCost();
                    end;
                }
                field(inventoryPostingGroup; Rec."Inventory Posting Group")
                {
                    Caption = 'Inventory Posting Group';
                    
                    trigger OnValidate()
                    begin
                        ValidateInventoryPostingGroup();
                    end;
                }
            }
        }
    }
    
    local procedure ValidateItemNumber()
    var
        ItemNumberPattern: Text;
        NoSeriesManagement: Codeunit NoSeriesManagement;
    begin
        if Rec."No." = '' then
            Error('Item number cannot be empty');
            
        // Validate item number format
        ItemNumberPattern := GetItemNumberPattern();
        if ItemNumberPattern <> '' then
            if not MatchesPattern(Rec."No.", ItemNumberPattern) then
                Error('Item number %1 does not match required pattern %2', Rec."No.", ItemNumberPattern);
                
        // Check for duplicate item numbers
        if ItemNumberExists(Rec."No.", Rec.SystemId) then
            Error('Item number %1 already exists', Rec."No.");
    end;
    
    local procedure ValidateDescription()
    begin
        if Rec.Description = '' then
            Error('Item description is required');
            
        if StrLen(Rec.Description) < 3 then
            Error('Item description must be at least 3 characters long');
            
        // Check for prohibited words
        if ContainsProhibitedWords(Rec.Description) then
            Error('Item description contains prohibited content');
    end;
    
    local procedure ValidateUnitPrice()
    begin
        if Rec."Unit Price" < 0 then
            Error('Unit price cannot be negative');
            
        // Validate against unit cost for margin checking
        if (Rec."Unit Cost" > 0) and (Rec."Unit Price" < Rec."Unit Cost") then
            if not Confirm('Unit price is below unit cost. Continue?') then
                Error('Unit price validation failed');
                
        // Check for unrealistic pricing
        if Rec."Unit Price" > 1000000 then
            Error('Unit price exceeds maximum allowed amount');
    end;
    
    local procedure ValidateUnitCost()
    begin
        if Rec."Unit Cost" < 0 then
            Error('Unit cost cannot be negative');
            
        // Validate cost against existing inventory value
        ValidateCostAgainstInventory();
    end;
    
    local procedure ValidateInventoryPostingGroup()
    var
        InventoryPostingGroup: Record "Inventory Posting Group";
    begin
        if Rec."Inventory Posting Group" = '' then
            Error('Inventory posting group is required');
            
        if not InventoryPostingGroup.Get(Rec."Inventory Posting Group") then
            Error('Inventory posting group %1 does not exist', Rec."Inventory Posting Group");
    end;
    
    local procedure GetItemNumberPattern(): Text
    var
        InventorySetup: Record "Inventory Setup";
    begin
        if InventorySetup.Get() then
            exit(InventorySetup."Item Nos.");  // Use number series as pattern
        exit('');
    end;
    
    local procedure MatchesPattern(Value: Text; Pattern: Text): Boolean
    var
        NoSeriesLine: Record "No. Series Line";
    begin
        // Simplified pattern matching for demo
        // In real implementation, use proper number series validation
        exit(StrLen(Value) >= 4);
    end;
    
    local procedure ItemNumberExists(ItemNo: Code[20]; ExcludeSystemId: Guid): Boolean
    var
        Item: Record Item;
    begin
        Item.SetRange("No.", ItemNo);
        Item.SetFilter(SystemId, '<>%1', ExcludeSystemId);
        exit(not Item.IsEmpty());
    end;
    
    local procedure ContainsProhibitedWords(Description: Text): Boolean
    var
        ProhibitedWords: List of [Text];
        Word: Text;
    begin
        // Define prohibited words for item descriptions
        ProhibitedWords.Add('TEMP');
        ProhibitedWords.Add('TEST');
        ProhibitedWords.Add('DELETE');
        
        foreach Word in ProhibitedWords do
            if Description.ToUpper().Contains(Word) then
                exit(true);
                
        exit(false);
    end;
    
    local procedure ValidateCostAgainstInventory()
    var
        ItemLedgerEntry: Record "Item Ledger Entry";
        AverageCost: Decimal;
        Variance: Decimal;
    begin
        // Calculate current average cost
        ItemLedgerEntry.SetRange("Item No.", Rec."No.");
        ItemLedgerEntry.CalcSums(Quantity, "Cost Amount (Actual)");
        
        if ItemLedgerEntry.Quantity <> 0 then begin
            AverageCost := ItemLedgerEntry."Cost Amount (Actual)" / ItemLedgerEntry.Quantity;
            Variance := Abs(Rec."Unit Cost" - AverageCost) / AverageCost * 100;
            
            // Warn if cost variance exceeds threshold
            if Variance > 25 then  // 25% variance threshold
                if not Confirm('Unit cost varies %1% from average inventory cost. Continue?', false, Round(Variance, 1)) then
                    Error('Cost validation failed');
        end;
    end;
}
```

## Cross-Field and Business Logic Validation

```al
// Complex validation codeunit for sales orders
codeunit 50501 "Sales Order Validation API"
{
    // Comprehensive sales order validation
    procedure ValidateSalesOrder(var SalesHeader: Record "Sales Header"): Boolean
    begin
        // Execute all validation rules in sequence
        if not ValidateOrderHeader(SalesHeader) then
            exit(false);
            
        if not ValidateCustomerRelationship(SalesHeader) then
            exit(false);
            
        if not ValidateOrderTotals(SalesHeader) then
            exit(false);
            
        if not ValidateInventoryAvailability(SalesHeader) then
            exit(false);
            
        exit(true);
    end;
    
    local procedure ValidateOrderHeader(var SalesHeader: Record "Sales Header"): Boolean
    begin
        // Date validations
        if SalesHeader."Order Date" = 0D then
            Error('Order date is required');
            
        if SalesHeader."Order Date" < WorkDate() - 30 then
            Error('Order date cannot be more than 30 days in the past');
            
        if SalesHeader."Requested Delivery Date" < SalesHeader."Order Date" then
            Error('Requested delivery date cannot be before order date');
            
        // Document status validation
        if SalesHeader."Document Type" <> SalesHeader."Document Type"::Order then
            Error('Only sales orders can be processed through this API');
            
        exit(true);
    end;
    
    local procedure ValidateCustomerRelationship(var SalesHeader: Record "Sales Header"): Boolean
    var
        Customer: Record Customer;
        CustLedgerEntry: Record "Cust. Ledger Entry";
        OutstandingAmount: Decimal;
    begin
        // Validate customer exists and is active
        if not Customer.Get(SalesHeader."Sell-to Customer No.") then
            Error('Customer %1 does not exist', SalesHeader."Sell-to Customer No.");
            
        if Customer.Blocked <> Customer.Blocked::" " then
            Error('Customer %1 is blocked for %2', Customer."No.", Customer.Blocked);
            
        // Check credit limit
        if Customer."Credit Limit (LCY)" > 0 then begin
            CustLedgerEntry.SetRange("Customer No.", Customer."No.");
            CustLedgerEntry.SetRange(Open, true);
            CustLedgerEntry.CalcSums("Remaining Amt. (LCY)");
            OutstandingAmount := CustLedgerEntry."Remaining Amt. (LCY)";
            
            SalesHeader.CalcFields("Amount Including VAT");
            if (OutstandingAmount + SalesHeader."Amount Including VAT") > Customer."Credit Limit (LCY)" then
                Error('Order would exceed customer credit limit. Outstanding: %1, Order: %2, Limit: %3', 
                      OutstandingAmount, SalesHeader."Amount Including VAT", Customer."Credit Limit (LCY)");
        end;
        
        exit(true);
    end;
    
    local procedure ValidateOrderTotals(var SalesHeader: Record "Sales Header"): Boolean
    var
        SalesLine: Record "Sales Line";
        TotalAmount: Decimal;
        LineCount: Integer;
    begin
        // Validate order has lines
        SalesLine.SetRange("Document Type", SalesHeader."Document Type");
        SalesLine.SetRange("Document No.", SalesHeader."No.");
        LineCount := SalesLine.Count();
        
        if LineCount = 0 then
            Error('Sales order must have at least one line');
            
        if LineCount > 100 then
            Error('Sales order cannot have more than 100 lines');
            
        // Validate minimum order amount
        SalesHeader.CalcFields("Amount Including VAT");
        if SalesHeader."Amount Including VAT" < 10 then
            Error('Minimum order amount is 10.00');
            
        exit(true);
    end;
    
    local procedure ValidateInventoryAvailability(var SalesHeader: Record "Sales Header"): Boolean
    var
        SalesLine: Record "Sales Line";
        Item: Record Item;
        AvailableInventory: Decimal;
    begin
        SalesLine.SetRange("Document Type", SalesHeader."Document Type");
        SalesLine.SetRange("Document No.", SalesHeader."No.");
        SalesLine.SetRange(Type, SalesLine.Type::Item);
        
        if SalesLine.FindSet() then
            repeat
                if Item.Get(SalesLine."No.") then begin
                    Item.CalcFields(Inventory, "Qty. on Sales Order");
                    AvailableInventory := Item.Inventory - Item."Qty. on Sales Order";
                    
                    if SalesLine.Quantity > AvailableInventory then
                        Error('Insufficient inventory for item %1. Requested: %2, Available: %3', 
                              Item."No.", SalesLine.Quantity, AvailableInventory);
                end;
            until SalesLine.Next() = 0;
            
        exit(true);
    end;
}

// API page with comprehensive validation
page 50502 "Sales Order Validation API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'sales';
    APIVersion = 'v1.0';
    EntityName = 'salesOrder';
    EntitySetName = 'salesOrders';
    SourceTable = "Sales Header";
    DelayedInsert = true;
    ODataKeyFields = SystemId;
    SourceTableView = WHERE("Document Type" = CONST(Order));
    
    layout
    {
        area(Content)
        {
            repeater(GroupName)
            {
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                    Editable = false;
                }
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                }
                field(customerNumber; Rec."Sell-to Customer No.")
                {
                    Caption = 'Customer Number';
                }
                field(orderDate; Rec."Order Date")
                {
                    Caption = 'Order Date';
                }
                field(requestedDeliveryDate; Rec."Requested Delivery Date")
                {
                    Caption = 'Requested Delivery Date';
                }
                field(totalAmount; TotalAmount)
                {
                    Caption = 'Total Amount';
                    Editable = false;
                }
            }
        }
    }
    
    var
        TotalAmount: Decimal;
        SalesOrderValidation: Codeunit "Sales Order Validation API";
    
    trigger OnAfterGetRecord()
    begin
        Rec.CalcFields("Amount Including VAT");
        TotalAmount := Rec."Amount Including VAT";
    end;
    
    trigger OnInsertRecord(BelowxRec: Boolean): Boolean
    begin
        // Execute comprehensive validation before insert
        exit(SalesOrderValidation.ValidateSalesOrder(Rec));
    end;
    
    trigger OnModifyRecord(): Boolean
    begin
        // Execute validation before modification
        exit(SalesOrderValidation.ValidateSalesOrder(Rec));
    end;
}
```

## Implementation Notes

**Validation Rule Execution Best Practices:**

1. **Validation Strategy:**
   - Implement field-level validation for immediate feedback
   - Use record-level validation for business logic enforcement
   - Combine BC standard validation with custom business rules
   - Provide clear, actionable error messages

2. **Performance Optimization:**
   - Execute lightweight validations first (required fields, formats)
   - Defer expensive validations (database lookups) until necessary
   - Cache validation results when possible
   - Use filtered queries to minimize database access

3. **Error Handling:**
   - Return specific error messages indicating the validation failure
   - Include relevant data in error messages for troubleshooting
   - Use consistent error message formats across API endpoints
   - Log validation failures for monitoring and improvement

4. **Business Rule Integration:**
   - Leverage existing BC table validation where possible
   - Implement custom validation for API-specific requirements
   - Ensure validation rules align with UI behavior
   - Document validation rules for API consumers