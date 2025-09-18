# API Page Implementation - AL Code Examples

## Basic API Page Structure

```al
page 50100 "Customer API"
{
    APIPublisher = 'contoso';
    APIGroup = 'business';
    APIVersion = 'v1.0';
    EntityName = 'customer';
    EntitySetName = 'customers';
    PageType = API;
    Caption = 'Customer API';
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
                }
                field(displayName; Rec.Name)
                {
                    Caption = 'Display Name';
                }
                field(type; Rec."Customer Posting Group")
                {
                    Caption = 'Type';
                }
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
}
```

## Business Entity Relationships

```al
page 50101 "Sales Order API"
{
    APIPublisher = 'contoso';
    APIGroup = 'sales';
    APIVersion = 'v2.0';
    EntityName = 'salesOrder';
    EntitySetName = 'salesOrders';
    PageType = API;
    Caption = 'Sales Order API';
    SourceTable = "Sales Header";
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
                }
                field(orderDate; Rec."Order Date")
                {
                    Caption = 'Order Date';
                }
                // Customer relationship - expose related entity
                field(customerId; Rec."Sell-to Customer No.")
                {
                    Caption = 'Customer Id';
                }
                field(customerName; Rec."Sell-to Customer Name")
                {
                    Caption = 'Customer Name';
                    Editable = false;
                }
                // Currency handling with proper OData exposure
                field(currencyCode; CurrencyCodeText)
                {
                    Caption = 'Currency Code';
                    trigger OnValidate()
                    begin
                        if CurrencyCodeText = '' then
                            Rec."Currency Code" := ''
                        else
                            Rec."Currency Code" := CopyStr(CurrencyCodeText, 1, MaxStrLen(Rec."Currency Code"));
                    end;

                    trigger OnAfterValidate()
                    begin
                        CurrencyCodeText := Rec."Currency Code";
                    end;
                }
                // Calculated fields for API consumption
                field(totalAmountExcludingTax; TotalAmountExclTax)
                {
                    Caption = 'Total Amount Excluding Tax';
                    Editable = false;
                }
                field(totalAmountIncludingTax; TotalAmountInclTax)
                {
                    Caption = 'Total Amount Including Tax';
                    Editable = false;
                }
                // Status enum exposure
                field(status; Rec.Status)
                {
                    Caption = 'Status';
                }
                // Navigation properties for related entities
                part(salesOrderLines; "Sales Order Line API")
                {
                    Caption = 'Lines';
                    EntityName = 'salesOrderLine';
                    EntitySetName = 'salesOrderLines';
                    SubPageLink = "Document No." = field("No."), "Document Type" = field("Document Type");
                }
            }
        }
    }

    var
        CurrencyCodeText: Text[10];
        TotalAmountExclTax: Decimal;
        TotalAmountInclTax: Decimal;

    trigger OnAfterGetRecord()
    begin
        CurrencyCodeText := Rec."Currency Code";
        CalculateTotals();
    end;

    local procedure CalculateTotals()
    var
        SalesLine: Record "Sales Line";
    begin
        SalesLine.SetRange("Document Type", Rec."Document Type");
        SalesLine.SetRange("Document No.", Rec."No.");
        SalesLine.CalcSums("Line Amount", "Amount Including VAT");
        TotalAmountExclTax := SalesLine."Line Amount";
        TotalAmountInclTax := SalesLine."Amount Including VAT";
    end;
}
```

## Security and Permission Integration

```al
page 50102 "Secure Customer API"
{
    APIPublisher = 'contoso';
    APIGroup = 'business';
    APIVersion = 'v1.0';
    EntityName = 'secureCustomer';
    EntitySetName = 'secureCustomers';
    PageType = API;
    Caption = 'Secure Customer API';
    SourceTable = Customer;
    DelayedInsert = true;
    ODataKeyFields = SystemId;
    // API-specific permissions
    Permissions = tabledata Customer = RIMD;

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
                field(displayName; Rec.Name)
                {
                    Caption = 'Display Name';
                }
                // Conditional field exposure based on permissions
                field(creditLimit; CreditLimitValue)
                {
                    Caption = 'Credit Limit';
                    trigger OnValidate()
                    begin
                        CheckCreditLimitPermission();
                        Rec."Credit Limit (LCY)" := CreditLimitValue;
                    end;
                }
                // Sensitive data with read-only access control
                field(balance; BalanceValue)
                {
                    Caption = 'Balance';
                    Editable = false;
                }
            }
        }
    }

    var
        CreditLimitValue: Decimal;
        BalanceValue: Decimal;

    trigger OnAfterGetRecord()
    begin
        // Apply security filters and data masking
        ApplySecurityFilters();
        LoadConditionalData();
    end;

    trigger OnInsertRecord(BelowxRec: Boolean): Boolean
    begin
        CheckInsertPermission();
        exit(true);
    end;

    trigger OnModifyRecord(): Boolean
    begin
        CheckModifyPermission();
        exit(true);
    end;

    trigger OnDeleteRecord(): Boolean
    begin
        CheckDeletePermission();
        exit(true);
    end;

    local procedure ApplySecurityFilters()
    var
        UserSetup: Record "User Setup";
    begin
        // Apply user-specific filtering
        if UserSetup.Get(UserId) then begin
            if UserSetup."Salespers./Purch. Code" <> '' then
                Rec.SetRange("Salesperson Code", UserSetup."Salespers./Purch. Code");
        end;
    end;

    local procedure LoadConditionalData()
    var
        UserPermissions: Codeunit "User Permissions";
    begin
        // Load sensitive data based on permissions
        if UserPermissions.IsSuper(UserSecurityId()) then begin
            CreditLimitValue := Rec."Credit Limit (LCY)";
            Rec.CalcFields(Balance);
            BalanceValue := Rec.Balance;
        end else begin
            CreditLimitValue := 0;
            BalanceValue := 0;
        end;
    end;

    local procedure CheckCreditLimitPermission()
    var
        UserPermissions: Codeunit "User Permissions";
    begin
        if not UserPermissions.IsSuper(UserSecurityId()) then
            Error('Insufficient permissions to modify credit limit');
    end;

    local procedure CheckInsertPermission()
    begin
        if not CheckTablePermission(Database::Customer, TablePermission::Insert) then
            Error('Insufficient permissions to create customers');
    end;

    local procedure CheckModifyPermission()
    begin
        if not CheckTablePermission(Database::Customer, TablePermission::Modify) then
            Error('Insufficient permissions to modify customers');
    end;

    local procedure CheckDeletePermission()
    begin
        if not CheckTablePermission(Database::Customer, TablePermission::Delete) then
            Error('Insufficient permissions to delete customers');
    end;

    local procedure CheckTablePermission(TableNo: Integer; Permission: TablePermission): Boolean
    var
        RecordRef: RecordRef;
    begin
        RecordRef.Open(TableNo);
        exit(RecordRef.ReadPermission() and
             ((Permission = TablePermission::Insert) and RecordRef.WritePermission()) or
             ((Permission = TablePermission::Modify) and RecordRef.WritePermission()) or
             ((Permission = TablePermission::Delete) and RecordRef.WritePermission()));
    end;
}

enum 50100 TablePermission
{
    Extensible = true;

    value(0; Read) { Caption = 'Read'; }
    value(1; Insert) { Caption = 'Insert'; }
    value(2; Modify) { Caption = 'Modify'; }
    value(3; Delete) { Caption = 'Delete'; }
}
```

## Performance Optimization Patterns

```al
page 50103 "Optimized Item API"
{
    APIPublisher = 'contoso';
    APIGroup = 'inventory';
    APIVersion = 'v1.0';
    EntityName = 'item';
    EntitySetName = 'items';
    PageType = API;
    Caption = 'Optimized Item API';
    SourceTable = Item;
    DelayedInsert = true;
    ODataKeyFields = SystemId;
    // Performance optimization: Define source table view
    SourceTableView = where(Blocked = const(false), Type = filter(<> "Non-Inventory"));

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
                field(displayName; Rec.Description)
                {
                    Caption = 'Display Name';
                }
                field(unitPrice; Rec."Unit Price")
                {
                    Caption = 'Unit Price';
                }
                // Efficient inventory calculation
                field(inventory; InventoryValue)
                {
                    Caption = 'Inventory';
                    Editable = false;
                }
                // Pre-calculated unit of measure relationship
                field(baseUnitOfMeasureCode; Rec."Base Unit of Measure")
                {
                    Caption = 'Base Unit Of Measure Code';
                    Editable = false;
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
        InventoryValue: Decimal;

    trigger OnAfterGetRecord()
    begin
        LoadOptimizedData();
    end;

    trigger OnOpenPage()
    begin
        SetOptimalFilters();
    end;

    local procedure LoadOptimizedData()
    begin
        // Efficient inventory calculation - avoid FlowField if possible
        if ShouldCalculateInventory() then begin
            Rec.CalcFields(Inventory);
            InventoryValue := Rec.Inventory;
        end else
            InventoryValue := 0;
    end;

    local procedure ShouldCalculateInventory(): Boolean
    begin
        // Conditional expensive calculations based on context
        exit(Rec.Type = Rec.Type::Inventory);
    end;

    local procedure SetOptimalFilters()
    begin
        // Performance optimization: Apply efficient filters
        Rec.SetFilter(Type, '<>%1', Rec.Type::"Non-Inventory");
        Rec.SetRange(Blocked, false);
    end;
}
```

## Anti-Pattern Examples (What NOT to Do)

```al
// ❌ ANTI-PATTERN: Poor field exposure and mapping
page 50199 "Bad Field Mapping API"
{
    APIPublisher = 'contoso';
    APIGroup = 'business';
    APIVersion = 'v1.0';
    EntityName = 'badCustomer';
    EntitySetName = 'badCustomers';
    PageType = API;
    SourceTable = Customer;
    // ❌ Missing: DelayedInsert = true for better performance
    // ❌ Missing: ODataKeyFields specification

    layout
    {
        area(Content)
        {
            repeater(Group)
            {
                // ❌ Wrong: Using database field names as API field names
                field("No."; Rec."No.") { }
                field("Customer Posting Group"; Rec."Customer Posting Group") { }

                // ❌ Wrong: No relationship handling for lookups
                field("Currency Code"; Rec."Currency Code")
                {
                    // ❌ Missing: Proper relationship mapping to Currency entity
                }

                // ❌ Wrong: Exposing sensitive data without security checks
                field("Credit Limit (LCY)"; Rec."Credit Limit (LCY)") { }

                // ❌ Wrong: Exposing FlowFields without consideration for performance
                field(Balance; Rec.Balance) { }

                // ❌ Wrong: No proper validation or error handling
                field("Payment Terms Code"; Rec."Payment Terms Code")
                {
                    trigger OnValidate()
                    begin
                        // ❌ No validation - could cause data integrity issues
                    end;
                }
            }
        }
    }

    // ❌ Missing: Proper trigger implementations
    // ❌ Missing: Security and permission checks
    // ❌ Missing: Error handling and validation
}

// ❌ ANTI-PATTERN: Performance-killing implementation
page 50198 "Performance Bad API"
{
    APIPublisher = 'contoso';
    APIGroup = 'business';
    APIVersion = 'v1.0';
    EntityName = 'slowCustomer';
    EntitySetName = 'slowCustomers';
    PageType = API;
    SourceTable = Customer;
    ODataKeyFields = SystemId;

    layout
    {
        area(Content)
        {
            repeater(Group)
            {
                field(id; Rec.SystemId) { }
                field(number; Rec."No.") { }
                field(name; Rec.Name) { }

                // ❌ Performance killer: Calculating complex data on every record
                field(totalSales; GetTotalSales())
                {
                    Caption = 'Total Sales';
                }

                // ❌ Performance killer: Multiple FlowFields
                field(balance; Rec.Balance) { }
                field(balanceLCY; Rec."Balance (LCY)") { }
                field(salesLCY; Rec."Sales (LCY)") { }

                // ❌ Performance killer: Related table lookups in field triggers
                field(salesPersonName; GetSalesPersonName())
                {
                    Caption = 'Sales Person Name';
                }
            }
        }
    }

    // ❌ Performance killers: Heavy calculations in field methods
    local procedure GetTotalSales(): Decimal
    var
        CustLedgerEntry: Record "Cust. Ledger Entry";
        TotalAmount: Decimal;
    begin
        // ❌ Expensive calculation on every record load
        CustLedgerEntry.SetRange("Customer No.", Rec."No.");
        CustLedgerEntry.CalcSums("Sales (LCY)");
        exit(CustLedgerEntry."Sales (LCY)");
    end;

    local procedure GetSalesPersonName(): Text
    var
        SalespersonPurchaser: Record "Salesperson/Purchaser";
    begin
        // ❌ Database lookup on every record display
        if SalespersonPurchaser.Get(Rec."Salesperson Code") then
            exit(SalespersonPurchaser.Name)
        else
            exit('');
    end;
}
```

## Correct Implementation Patterns

```al
// ✅ CORRECT: Optimized API with proper structure
page 50197 "Optimized Customer API"
{
    APIPublisher = 'contoso';
    APIGroup = 'business';
    APIVersion = 'v1.0';
    EntityName = 'customer';
    EntitySetName = 'customers';
    PageType = API;
    Caption = 'Customer API';
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
                }
                field(displayName; Rec.Name)
                {
                    Caption = 'Display Name';
                }
                field(email; Rec."E-Mail")
                {
                    Caption = 'Email';
                }
                field(phoneNumber; Rec."Phone No.")
                {
                    Caption = 'Phone Number';
                }
                field(blocked; Rec.Blocked)
                {
                    Caption = 'Blocked';
                }
                // ✅ Calculated field with efficient loading
                field(salesPersonName; SalesPersonNameVar)
                {
                    Caption = 'Sales Person Name';
                    Editable = false;
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
        SalesPersonNameVar: Text[50];

    trigger OnAfterGetRecord()
    begin
        LoadRelatedData();
    end;

    local procedure LoadRelatedData()
    var
        SalespersonPurchaser: Record "Salesperson/Purchaser";
    begin
        // ✅ Efficient: Single lookup per record
        if Rec."Salesperson Code" <> '' then begin
            if SalespersonPurchaser.Get(Rec."Salesperson Code") then
                SalesPersonNameVar := SalespersonPurchaser.Name
            else
                SalesPersonNameVar := '';
        end else
            SalesPersonNameVar := '';
    end;
}
```

## API Implementation Guidelines

### Essential API Page Configuration
1. **Required Properties**: APIPublisher, APIGroup, APIVersion, EntityName, EntitySetName, PageType = API
2. **Performance Settings**: DelayedInsert = true, ODataKeyFields = SystemId
3. **Field Naming**: Use camelCase for API field names, provide meaningful captions
4. **Relationship Handling**: Use SystemId for entity relationships, implement proper validation

### Security Best Practices
1. **Permission Integration**: Use Permissions property and implement trigger-based permission checks
2. **Data Filtering**: Apply user-specific filters in OnAfterGetRecord trigger
3. **Sensitive Data**: Implement conditional loading based on user permissions
4. **Validation**: Add proper validation in field triggers and record triggers

### Performance Optimization
1. **Avoid FlowFields**: Calculate values in triggers instead of using FlowFields directly
2. **Efficient Queries**: Use SourceTableView to apply optimal filters
3. **Batch Loading**: Load related data efficiently in OnAfterGetRecord
4. **Conditional Processing**: Only perform expensive operations when necessary

### Error Handling
1. **Validation Errors**: Provide clear, actionable error messages
2. **Permission Errors**: Return appropriate HTTP status codes for authorization issues
3. **Business Logic**: Respect Business Central validation rules and constraints
4. **Data Integrity**: Ensure API operations maintain data consistency