# API Page Field Control Selection Strategy - AL Code Sample

## Basic Field Control Selection

```al
page 50110 "Item API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'inventory';
    APIVersion = 'v1.0';
    EntityName = 'item';
    EntitySetName = 'items';
    SourceTable = Item;
    DelayedInsert = true;
    
    layout
    {
        area(Content)
        {
            repeater(Records)
            {
                // Primary identifier - always include SystemId
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                    Editable = false;  // System-generated, read-only
                }
                
                // Business key - essential for integration
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                    // Editable on insert, read-only after
                    Editable = Rec."No." = '';
                }
                
                // Core business fields - commonly queried
                field(displayName; Rec.Description)
                {
                    Caption = 'Display Name';
                }
                
                field(type; Rec.Type)
                {
                    Caption = 'Type';
                }
                
                field(unitPrice; Rec."Unit Price")
                {
                    Caption = 'Unit Price';
                }
                
                // Inventory tracking - frequently needed
                field(inventory; Rec.Inventory)
                {
                    Caption = 'Inventory';
                    Editable = false;  // Calculated field
                }
                
                // Status field - important for business logic
                field(blocked; Rec.Blocked)
                {
                    Caption = 'Blocked';
                }
                
                // Timestamps - useful for synchronization
                field(lastModifiedDateTime; Rec.SystemModifiedAt)
                {
                    Caption = 'Last Modified Date Time';
                    Editable = false;
                }
            }
        }
    }
}
```

## Advanced Field Selection with Business Logic

```al
page 50111 "Customer API Advanced"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'customers';
    APIVersion = 'v2.0';
    EntityName = 'customer';
    EntitySetName = 'customers';
    SourceTable = Customer;
    DelayedInsert = true;
    
    layout
    {
        area(Content)
        {
            repeater(Records)
            {
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                }
                
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                    trigger OnValidate()
                    begin
                        // Custom validation for API
                        if StrLen(Rec."No.") < 3 then
                            Error('Customer number must be at least 3 characters');
                    end;
                }
                
                field(displayName; Rec.Name)
                {
                    Caption = 'Display Name';
                    NotBlank = true;  // Required field enforcement
                }
                
                // Complex field with business logic
                field(creditLimit; Rec."Credit Limit (LCY)")
                {
                    Caption = 'Credit Limit';
                    trigger OnValidate()
                    begin
                        // Business rule: Credit limit validation
                        if Rec."Credit Limit (LCY)" < 0 then
                            Error('Credit limit cannot be negative');
                            
                        if Rec."Credit Limit (LCY)" > 1000000 then
                            if not ConfirmManagement.GetResponseOrDefault('Credit limit exceeds 1M. Continue?', false) then
                                Error('Credit limit validation failed');
                    end;
                }
                
                // Calculated field not stored in table
                field(currentBalance; CurrentBalance)
                {
                    Caption = 'Current Balance';
                    Editable = false;
                    
                    trigger OnLookup(var Text: Text): Boolean
                    begin
                        // Navigate to customer ledger entries
                        CustLedgEntry.SetRange("Customer No.", Rec."No.");
                        if Page.RunModal(Page::"Customer Ledger Entries", CustLedgEntry) = Action::LookupOK then
                            exit(true);
                    end;
                }
                
                // Address fields - grouped for related data
                field(addressLine1; Rec.Address)
                {
                    Caption = 'Address Line 1';
                }
                
                field(addressLine2; Rec."Address 2")
                {
                    Caption = 'Address Line 2';
                }
                
                field(city; Rec.City)
                {
                    Caption = 'City';
                }
                
                field(state; Rec.County)
                {
                    Caption = 'State';
                }
                
                field(postalCode; Rec."Post Code")
                {
                    Caption = 'Postal Code';
                    trigger OnValidate()
                    begin
                        // Postal code format validation
                        ValidatePostalCode(Rec."Post Code", Rec."Country/Region Code");
                    end;
                }
                
                field(countryCode; Rec."Country/Region Code")
                {
                    Caption = 'Country Code';
                }
                
                // Contact information
                field(phoneNumber; Rec."Phone No.")
                {
                    Caption = 'Phone Number';
                    ExtendedDatatype = PhoneNo;
                }
                
                field(email; Rec."E-Mail")
                {
                    Caption = 'Email';
                    ExtendedDatatype = EMail;
                }
                
                // Financial fields
                field(paymentTermsId; PaymentTermsId)
                {
                    Caption = 'Payment Terms Id';
                    
                    trigger OnValidate()
                    begin
                        ValidatePaymentTermsId();
                    end;
                    
                    trigger OnLookup(var Text: Text): Boolean
                    begin
                        exit(LookupPaymentTerms(Text));
                    end;
                }
                
                field(paymentMethodId; PaymentMethodId)
                {
                    Caption = 'Payment Method Id';
                }
                
                // Status and control fields
                field(blocked; Rec.Blocked)
                {
                    Caption = 'Blocked';
                }
                
                field(lastModifiedDateTime; Rec.SystemModifiedAt)
                {
                    Caption = 'Last Modified Date Time';
                    Editable = false;
                }
            }
        }
    }
    
    var
        CustLedgEntry: Record "Cust. Ledger Entry";
        ConfirmManagement: Codeunit "Confirm Management";
        CurrentBalance: Decimal;
        PaymentTermsId: Guid;
        PaymentMethodId: Guid;
    
    trigger OnAfterGetRecord()
    begin
        // Calculate current balance for each record
        CalcCurrentBalance();
        GetPaymentTermsId();
        GetPaymentMethodId();
    end;
    
    local procedure CalcCurrentBalance()
    begin
        Rec.CalcFields("Balance (LCY)");
        CurrentBalance := Rec."Balance (LCY)";
    end;
    
    local procedure ValidatePostalCode(PostalCode: Code[20]; CountryCode: Code[10])
    var
        PostCodeRegex: Codeunit Regex;
    begin
        // Example validation for US postal codes
        if CountryCode = 'US' then
            if not PostCodeRegex.IsMatch(PostalCode, '^\d{5}(-\d{4})?$') then
                Error('Invalid US postal code format');
    end;
    
    local procedure ValidatePaymentTermsId()
    var
        PaymentTerms: Record "Payment Terms";
    begin
        if IsNullGuid(PaymentTermsId) then
            Rec."Payment Terms Code" := ''
        else begin
            PaymentTerms.SetRange(SystemId, PaymentTermsId);
            if PaymentTerms.FindFirst() then
                Rec."Payment Terms Code" := PaymentTerms.Code
            else
                Error('Payment terms not found for specified ID');
        end;
    end;
}
```

## Field Selection for Different API Scenarios

```al
// Read-only reporting API - minimal field set
page 50112 "Sales Statistics API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'reporting';
    APIVersion = 'v1.0';
    EntityName = 'salesStatistic';
    EntitySetName = 'salesStatistics';
    SourceTable = Customer;
    DelayedInsert = true;
    Editable = false;  // Read-only API
    
    layout
    {
        area(Content)
        {
            repeater(Records)
            {
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                }
                
                field(customerNumber; Rec."No.")
                {
                    Caption = 'Customer Number';
                }
                
                field(customerName; Rec.Name)
                {
                    Caption = 'Customer Name';
                }
                
                // Aggregated sales data - calculated fields only
                field(totalSalesYTD; TotalSalesYTD)
                {
                    Caption = 'Total Sales YTD';
                }
                
                field(totalSalesLastYear; TotalSalesLastYear)
                {
                    Caption = 'Total Sales Last Year';
                }
                
                field(averageOrderValue; AverageOrderValue)
                {
                    Caption = 'Average Order Value';
                }
                
                field(orderCount; OrderCount)
                {
                    Caption = 'Order Count';
                }
                
                field(lastOrderDate; LastOrderDate)
                {
                    Caption = 'Last Order Date';
                }
            }
        }
    }
    
    var
        TotalSalesYTD: Decimal;
        TotalSalesLastYear: Decimal;
        AverageOrderValue: Decimal;
        OrderCount: Integer;
        LastOrderDate: Date;
    
    trigger OnAfterGetRecord()
    begin
        CalculateSalesStatistics();
    end;
    
    local procedure CalculateSalesStatistics()
    var
        SalesHeader: Record "Sales Header";
        SalesLine: Record "Sales Line";
    begin
        // Calculate YTD sales
        SalesLine.SetRange("Sell-to Customer No.", Rec."No.");
        SalesLine.SetRange("Document Type", SalesLine."Document Type"::Order);
        SalesLine.SetFilter("Shipment Date", '%1..%2', CalcDate('<-CY>', Today), Today);
        SalesLine.CalcSums("Line Amount");
        TotalSalesYTD := SalesLine."Line Amount";
        
        // Calculate last year sales
        SalesLine.SetFilter("Shipment Date", '%1..%2', CalcDate('<-1Y-CY>', Today), CalcDate('<-CY>', Today));
        SalesLine.CalcSums("Line Amount");
        TotalSalesLastYear := SalesLine."Line Amount";
        
        // Calculate order statistics
        SalesHeader.SetRange("Sell-to Customer No.", Rec."No.");
        SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
        OrderCount := SalesHeader.Count;
        
        if OrderCount > 0 then
            AverageOrderValue := TotalSalesYTD / OrderCount;
            
        SalesHeader.SetCurrentKey("Order Date");
        SalesHeader.Ascending(false);
        if SalesHeader.FindFirst() then
            LastOrderDate := SalesHeader."Order Date";
    end;
}
```

## Dynamic Field Selection Based on Context

```al
// Context-aware field selection
page 50113 "Item Context API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'inventory';
    APIVersion = 'v2.0';
    EntityName = 'item';
    EntitySetName = 'items';
    SourceTable = Item;
    DelayedInsert = true;
    
    layout
    {
        area(Content)
        {
            repeater(Records)
            {
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                }
                
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                }
                
                field(description; Rec.Description)
                {
                    Caption = 'Description';
                }
                
                // Conditionally visible based on item type
                field(inventoryPostingGroup; Rec."Inventory Posting Group")
                {
                    Caption = 'Inventory Posting Group';
                    Visible = Rec.Type = Rec.Type::Inventory;
                }
                
                field(unitCost; Rec."Unit Cost")
                {
                    Caption = 'Unit Cost';
                    Visible = ShowCostFields;
                }
                
                field(lastDirectCost; Rec."Last Direct Cost")
                {
                    Caption = 'Last Direct Cost';
                    Visible = ShowCostFields;
                }
                
                // Service-specific fields
                field(serviceItemGroup; Rec."Service Item Group")
                {
                    Caption = 'Service Item Group';
                    Visible = Rec.Type = Rec.Type::"Service";
                }
                
                // Manufacturing fields
                field(manufacturingPolicy; Rec."Manufacturing Policy")
                {
                    Caption = 'Manufacturing Policy';
                    Visible = ShowManufacturingFields;
                }
                
                field(replenishmentSystem; Rec."Replenishment System")
                {
                    Caption = 'Replenishment System';
                    Visible = ShowManufacturingFields;
                }
                
                // Conditionally calculated fields
                field(availableInventory; AvailableInventory)
                {
                    Caption = 'Available Inventory';
                    Visible = Rec.Type = Rec.Type::Inventory;
                }
            }
        }
    }
    
    var
        ShowCostFields: Boolean;
        ShowManufacturingFields: Boolean;
        AvailableInventory: Decimal;
    
    trigger OnOpenPage()
    begin
        // Determine field visibility based on user permissions or setup
        ShowCostFields := HasCostPermissions();
        ShowManufacturingFields := HasManufacturingModule();
    end;
    
    trigger OnAfterGetRecord()
    begin
        if Rec.Type = Rec.Type::Inventory then
            CalculateAvailableInventory();
    end;
    
    local procedure HasCostPermissions(): Boolean
    var
        UserSetup: Record "User Setup";
    begin
        if UserSetup.Get(UserId) then
            exit(UserSetup."View Cost Information");
        exit(false);
    end;
    
    local procedure HasManufacturingModule(): Boolean
    var
        ManufacturingSetup: Record "Manufacturing Setup";
    begin
        exit(ManufacturingSetup.Get());
    end;
    
    local procedure CalculateAvailableInventory()
    begin
        Rec.CalcFields(Inventory, "Reserved Qty. on Inventory");
        AvailableInventory := Rec.Inventory - Rec."Reserved Qty. on Inventory";
    end;
}
```

## Implementation Notes

**Field Selection Principles:**
- Always include SystemId for unique identification
- Include business keys (No., Code) for integration
- Select fields based on API purpose (CRUD vs reporting)
- Consider performance impact of calculated fields
- Group related fields together logically

**Validation and Business Logic:**
- Implement field-level validation in triggers
- Use NotBlank for required fields
- Handle complex validations in OnValidate triggers
- Consider API-specific validation rules

**Performance Considerations:**
- Avoid FlowFields in high-volume scenarios
- Calculate complex fields only when needed
- Use appropriate field visibility to reduce payload
- Consider caching calculated values

**Security and Permissions:**
- Hide sensitive fields based on user permissions
- Implement field-level security as needed
- Consider data classification requirements
- Validate user access to restricted information