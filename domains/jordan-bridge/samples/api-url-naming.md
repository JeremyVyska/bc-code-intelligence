# API Page URL Structure and Entity Naming Patterns - AL Code Sample

## Basic API Page URL Structure

```al
page 50100 "Customer API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'customers';
    APIVersion = 'v1.0';
    EntityName = 'customer';
    EntitySetName = 'customers';
    SourceTable = Customer;
    DelayedInsert = true;
    
    // Resulting URL: /v2.0/companies({companyId})/contoso/customers/v1.0/customers
    
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
                field(displayName; Rec.Name)
                {
                    Caption = 'Display Name';
                }
            }
        }
    }
}
```

## Advanced API Naming with Business Context

```al
page 50101 "Sales Order Lines API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'sales';
    APIVersion = 'v2.0';
    EntityName = 'salesOrderLine';
    EntitySetName = 'salesOrderLines';
    SourceTable = "Sales Line";
    DelayedInsert = true;
    
    // URL: /v2.0/companies({companyId})/contoso/sales/v2.0/salesOrderLines
    // Business context: Sales domain, order line entities
    
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
                field(documentId; Rec."Document No.")
                {
                    Caption = 'Document Id';
                }
                field(lineNumber; Rec."Line No.")
                {
                    Caption = 'Line Number';
                }
                field(itemNumber; Rec."No.")
                {
                    Caption = 'Item Number';
                    trigger OnValidate()
                    begin
                        // Validation logic for item number
                        if Rec.Type <> Rec.Type::Item then
                            Error('Only items are supported in this API');
                    end;
                }
                field(quantity; Rec.Quantity)
                {
                    Caption = 'Quantity';
                }
                field(unitPrice; Rec."Unit Price")
                {
                    Caption = 'Unit Price';
                }
            }
        }
    }
}
```

## API Naming Conventions Best Practices

```al
// Example: Financial reporting API with proper naming hierarchy
page 50102 "GL Account Balances API"
{
    PageType = API;
    APIPublisher = 'contoso';           // Company/publisher identifier
    APIGroup = 'financialReporting';    // Business domain grouping
    APIVersion = 'v1.0';                // Version for breaking changes
    EntityName = 'glAccountBalance';    // Singular, camelCase entity
    EntitySetName = 'glAccountBalances'; // Plural entity set
    SourceTable = "G/L Account";
    DelayedInsert = true;
    
    // Naming Pattern Analysis:
    // - Publisher: Short, unique identifier for your organization
    // - Group: Business domain (sales, purchasing, financialReporting)
    // - Version: Semantic versioning (v1.0, v2.0)
    // - EntityName: Singular, descriptive, camelCase
    // - EntitySetName: Plural of EntityName
    
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
                field(name; Rec.Name)
                {
                    Caption = 'Name';
                }
                field(accountType; Rec."Account Type")
                {
                    Caption = 'Account Type';
                }
                // Calculated field for current balance
                field(currentBalance; CurrentBalance)
                {
                    Caption = 'Current Balance';
                }
            }
        }
    }
    
    var
        CurrentBalance: Decimal;
    
    trigger OnAfterGetRecord()
    begin
        // Calculate balance for each account
        Rec.CalcFields("Net Change");
        CurrentBalance := Rec."Net Change";
    end;
}
```

## Complex API URL Hierarchies

```al
// Parent-child relationship API with hierarchical URLs
page 50103 "Sales Order Headers API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'sales';
    APIVersion = 'v2.0';
    EntityName = 'salesOrder';
    EntitySetName = 'salesOrders';
    SourceTable = "Sales Header";
    DelayedInsert = true;
    
    // Base URL: /contoso/sales/v2.0/salesOrders
    
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
                field(customerNumber; Rec."Sell-to Customer No.")
                {
                    Caption = 'Customer Number';
                }
                field(orderDate; Rec."Order Date")
                {
                    Caption = 'Order Date';
                }
                
                // Navigation property to lines
                part(salesOrderLines; "Sales Order Lines Sub API")
                {
                    Caption = 'Sales Order Lines';
                    EntityName = 'salesOrderLine';
                    EntitySetName = 'salesOrderLines';
                    SubPageLink = "Document No." = field("No."),
                                  "Document Type" = field("Document Type");
                }
            }
        }
    }
}

// Child API page for navigation
page 50104 "Sales Order Lines Sub API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'sales';
    APIVersion = 'v2.0';
    EntityName = 'salesOrderLine';
    EntitySetName = 'salesOrderLines';
    SourceTable = "Sales Line";
    DelayedInsert = true;
    
    // Accessed via: /contoso/sales/v2.0/salesOrders({id})/salesOrderLines
    
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
                field(lineNumber; Rec."Line No.")
                {
                    Caption = 'Line Number';
                }
                field(itemNumber; Rec."No.")
                {
                    Caption = 'Item Number';
                }
                field(description; Rec.Description)
                {
                    Caption = 'Description';
                }
                field(quantity; Rec.Quantity)
                {
                    Caption = 'Quantity';
                }
            }
        }
    }
}
```

## Implementation Notes

**URL Structure Components:**
- Base URL: `/v2.0/companies({companyId})`
- Publisher: Unique identifier for your organization
- Group: Business domain logical grouping
- Version: API version for managing breaking changes
- Entity Set: Plural noun for the collection

**Naming Best Practices:**
- Use camelCase for entity and field names
- Keep names descriptive but concise
- Avoid abbreviations unless universally understood
- Group related APIs under same APIGroup
- Use semantic versioning for APIVersion

**Performance Considerations:**
- EntityName and EntitySetName affect OData metadata generation
- Proper naming improves API discoverability
- Consistent naming patterns reduce integration complexity
- Consider future extensibility when choosing names