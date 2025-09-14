# SystemId Integration in API Pages - AL Code Sample

## Basic SystemId Usage as Primary Key

```al
page 50200 "Customer SystemId API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'customers';
    APIVersion = 'v1.0';
    EntityName = 'customer';
    EntitySetName = 'customers';
    SourceTable = Customer;
    DelayedInsert = true;
    ODataKeyFields = SystemId;  // Use SystemId as primary OData key
    
    layout
    {
        area(Content)
        {
            repeater(GroupName)
            {
                // SystemId exposed as 'id' for REST conventions
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                    Editable = false;  // SystemId is system-generated
                }
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                    // Business key still available for reference
                }
                field(displayName; Rec.Name)
                {
                    Caption = 'Display Name';
                }
                field(lastModifiedDateTime; Rec.SystemModifiedAt)
                {
                    Caption = 'Last Modified Date Time';
                    Editable = false;
                }
            }
        }
    }
    
    // SystemId automatically handled by BC platform
    // No custom triggers needed for SystemId generation
}
```

## SystemId with Related Records Navigation

```al
page 50201 "Sales Order SystemId API"
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
                field(customerId; CustomerSystemId)
                {
                    Caption = 'Customer Id';
                    // Link to customer using SystemId instead of business key
                }
                field(customerNumber; Rec."Sell-to Customer No.")
                {
                    Caption = 'Customer Number';
                    // Keep business key for backward compatibility
                }
                field(orderDate; Rec."Order Date")
                {
                    Caption = 'Order Date';
                }
                field(totalAmount; TotalAmount)
                {
                    Caption = 'Total Amount';
                }
            }
        }
    }
    
    var
        CustomerSystemId: Guid;
        TotalAmount: Decimal;
    
    trigger OnAfterGetRecord()
    var
        Customer: Record Customer;
    begin
        // Resolve customer SystemId from business key
        if Customer.Get(Rec."Sell-to Customer No.") then
            CustomerSystemId := Customer.SystemId
        else
            Clear(CustomerSystemId);
            
        // Calculate total amount
        Rec.CalcFields("Amount Including VAT");
        TotalAmount := Rec."Amount Including VAT";
    end;
    
    trigger OnModifyRecord(): Boolean
    var
        Customer: Record Customer;
    begin
        // Handle customer assignment via SystemId
        if not IsNullGuid(CustomerSystemId) then begin
            Customer.SetRange(SystemId, CustomerSystemId);
            if Customer.FindFirst() then
                Rec.Validate("Sell-to Customer No.", Customer."No.");
        end;
        exit(true);
    end;
}
```

## SystemId Cross-Reference Implementation

```al
// Helper codeunit for SystemId operations
codeunit 50200 "SystemId Helper"
{
    // Find record by SystemId across different tables
    procedure FindRecordBySystemId(TableID: Integer; SystemIdValue: Guid; var RecRef: RecordRef): Boolean
    var
        SystemIdFieldRef: FieldRef;
    begin
        RecRef.Open(TableID);
        SystemIdFieldRef := RecRef.Field(RecRef.SystemIdNo);
        SystemIdFieldRef.SetRange(SystemIdValue);
        exit(RecRef.FindFirst());
    end;
    
    // Get SystemId from business key
    procedure GetSystemIdFromBusinessKey(TableID: Integer; KeyFieldNo: Integer; KeyValue: Variant): Guid
    var
        RecRef: RecordRef;
        KeyFieldRef: FieldRef;
        SystemIdFieldRef: FieldRef;
    begin
        RecRef.Open(TableID);
        KeyFieldRef := RecRef.Field(KeyFieldNo);
        KeyFieldRef.SetRange(KeyValue);
        if RecRef.FindFirst() then begin
            SystemIdFieldRef := RecRef.Field(RecRef.SystemIdNo);
            exit(SystemIdFieldRef.Value);
        end;
        exit(CreateGuid());  // Return empty GUID if not found
    end;
    
    // Validate SystemId exists in target table
    procedure ValidateSystemIdExists(TableID: Integer; SystemIdValue: Guid): Boolean
    var
        RecRef: RecordRef;
    begin
        exit(FindRecordBySystemId(TableID, SystemIdValue, RecRef));
    end;
}

// API page demonstrating SystemId cross-references
page 50202 "Purchase Order Line API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'purchasing';
    APIVersion = 'v1.0';
    EntityName = 'purchaseOrderLine';
    EntitySetName = 'purchaseOrderLines';
    SourceTable = "Purchase Line";
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
                field(documentId; DocumentSystemId)
                {
                    Caption = 'Document Id';
                    // Reference to purchase header via SystemId
                }
                field(itemId; ItemSystemId)
                {
                    Caption = 'Item Id';
                    // Reference to item via SystemId
                }
                field(lineNumber; Rec."Line No.")
                {
                    Caption = 'Line Number';
                }
                field(quantity; Rec.Quantity)
                {
                    Caption = 'Quantity';
                }
                field(unitCost; Rec."Unit Cost")
                {
                    Caption = 'Unit Cost';
                }
            }
        }
    }
    
    var
        DocumentSystemId: Guid;
        ItemSystemId: Guid;
        SystemIdHelper: Codeunit "SystemId Helper";
    
    trigger OnAfterGetRecord()
    var
        PurchaseHeader: Record "Purchase Header";
        Item: Record Item;
    begin
        // Get document SystemId
        if PurchaseHeader.Get(Rec."Document Type", Rec."Document No.") then
            DocumentSystemId := PurchaseHeader.SystemId;
            
        // Get item SystemId
        if Item.Get(Rec."No.") then
            ItemSystemId := Item.SystemId;
    end;
    
    trigger OnModifyRecord(): Boolean
    var
        PurchaseHeader: Record "Purchase Header";
        Item: Record Item;
    begin
        // Validate and set document reference
        if not IsNullGuid(DocumentSystemId) then begin
            PurchaseHeader.SetRange(SystemId, DocumentSystemId);
            if PurchaseHeader.FindFirst() then begin
                Rec.Validate("Document Type", PurchaseHeader."Document Type");
                Rec.Validate("Document No.", PurchaseHeader."No.");
            end else
                Error('Purchase document with SystemId %1 not found', DocumentSystemId);
        end;
        
        // Validate and set item reference
        if not IsNullGuid(ItemSystemId) then begin
            Item.SetRange(SystemId, ItemSystemId);
            if Item.FindFirst() then
                Rec.Validate("No.", Item."No.")
            else
                Error('Item with SystemId %1 not found', ItemSystemId);
        end;
        
        exit(true);
    end;
}
```

## SystemId URL Navigation Patterns

```al
// API page with SystemId-based navigation URLs
page 50203 "Enhanced Customer API"
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
                }
                field(displayName; Rec.Name)
                {
                    Caption = 'Display Name';
                }
                field(salesOrdersUrl; SalesOrdersNavigationUrl)
                {
                    Caption = 'Sales Orders URL';
                    Editable = false;
                    // Navigation URL using SystemId
                }
                field(paymentsUrl; PaymentsNavigationUrl)
                {
                    Caption = 'Payments URL';
                    Editable = false;
                }
            }
        }
    }
    
    var
        SalesOrdersNavigationUrl: Text;
        PaymentsNavigationUrl: Text;
    
    trigger OnAfterGetRecord()
    begin
        // Build navigation URLs using SystemId
        SalesOrdersNavigationUrl := BuildNavigationUrl('salesOrders', Rec.SystemId);
        PaymentsNavigationUrl := BuildNavigationUrl('customerPayments', Rec.SystemId);
    end;
    
    local procedure BuildNavigationUrl(EntitySet: Text; RelatedSystemId: Guid): Text
    var
        BaseUrl: Text;
        CompanyName: Text;
    begin
        // Get base API URL from current request context
        BaseUrl := GetUrl(ClientType::ODataV4, CompanyName(), ObjectType::Page, 0);
        
        // Build related entity URL with SystemId filter
        exit(StrSubstNo('%1/%2?$filter=customerId eq %3', 
                       BaseUrl, EntitySet, RelatedSystemId));
    end;
}

// Related sales orders API filtered by customer SystemId
page 50204 "Customer Sales Orders API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'sales';
    APIVersion = 'v1.0';
    EntityName = 'customerSalesOrder';
    EntitySetName = 'customerSalesOrders';
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
                field(customerId; CustomerSystemId)
                {
                    Caption = 'Customer Id';
                    Editable = false;
                }
                field(orderNumber; Rec."No.")
                {
                    Caption = 'Order Number';
                }
                field(orderDate; Rec."Order Date")
                {
                    Caption = 'Order Date';
                }
            }
        }
    }
    
    var
        CustomerSystemId: Guid;
    
    trigger OnAfterGetRecord()
    var
        Customer: Record Customer;
    begin
        // Resolve customer SystemId
        if Customer.Get(Rec."Sell-to Customer No.") then
            CustomerSystemId := Customer.SystemId;
    end;
}
```

## Implementation Notes

**SystemId Integration Best Practices:**

1. **Primary Key Strategy:**
   - Always use SystemId as ODataKeyFields for API pages
   - Expose SystemId as 'id' field following REST conventions
   - Keep business keys available for backward compatibility

2. **Cross-Reference Management:**
   - Use SystemId for all entity relationships in APIs
   - Implement validation for SystemId references before updates
   - Provide helper functions for SystemId to business key conversion

3. **Navigation Patterns:**
   - Build related entity URLs using SystemId parameters
   - Implement consistent URL structures across API endpoints
   - Use SystemId filtering for efficient related data queries

4. **Performance Considerations:**
   - SystemId fields are automatically indexed by BC platform
   - Use SystemId for faster record lookups in large tables
   - Cache SystemId mappings for frequently accessed relationships