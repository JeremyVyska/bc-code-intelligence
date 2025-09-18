# ETag Implementation for Optimistic Concurrency - AL Code Sample

## Basic ETag Implementation

```al
page 50300 "Customer ETag API"
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
                field(lastModifiedDateTime; Rec.SystemModifiedAt)
                {
                    Caption = 'Last Modified Date Time';
                    Editable = false;
                    // Used for ETag generation
                }
                // ETag field for optimistic concurrency control
                field(etag; ETagValue)
                {
                    Caption = 'ETag';
                    Editable = false;
                }
            }
        }
    }
    
    var
        ETagValue: Text;
        OriginalETagValue: Text;
    
    trigger OnAfterGetRecord()
    begin
        // Generate ETag based on SystemModifiedAt
        ETagValue := GenerateETag(Rec.SystemModifiedAt, Rec.SystemId);
    end;
    
    trigger OnModifyRecord(): Boolean
    var
        CurrentRecord: Record Customer;
        CurrentETag: Text;
    begin
        // Validate ETag for optimistic concurrency
        if CurrentRecord.GetBySystemId(Rec.SystemId) then begin
            CurrentETag := GenerateETag(CurrentRecord.SystemModifiedAt, CurrentRecord.SystemId);
            
            // Check if record was modified by another user
            if CurrentETag <> OriginalETagValue then
                Error('The record has been modified by another user. Please refresh and try again.');
        end;
        
        exit(true);
    end;
    
    local procedure GenerateETag(ModifiedDateTime: DateTime; RecordSystemId: Guid): Text
    var
        TypeHelper: Codeunit "Type Helper";
        ETagSource: Text;
    begin
        // Combine timestamp and SystemId for unique ETag
        ETagSource := Format(ModifiedDateTime, 0, 9) + Format(RecordSystemId);
        exit(TypeHelper.GetMD5Hash(ETagSource));
    end;
}
```

## Advanced ETag with Custom Versioning

```al
// Custom versioning table for ETag management
table 50300 "Record Version Control"
{
    Caption = 'Record Version Control';
    DataClassification = SystemMetadata;
    
    fields
    {
        field(1; "Table ID"; Integer)
        {
            Caption = 'Table ID';
        }
        field(2; "Record SystemId"; Guid)
        {
            Caption = 'Record SystemId';
        }
        field(3; "Version Number"; Integer)
        {
            Caption = 'Version Number';
            InitValue = 1;
        }
        field(4; "Last Modified DateTime"; DateTime)
        {
            Caption = 'Last Modified DateTime';
        }
        field(5; "Modified By User"; Text[100])
        {
            Caption = 'Modified By User';
        }
        field(6; "ETag Value"; Text[100])
        {
            Caption = 'ETag Value';
        }
    }
    
    keys
    {
        key(PK; "Table ID", "Record SystemId")
        {
            Clustered = true;
        }
        key(ETag; "ETag Value")
        {
            // Index for fast ETag lookups
        }
    }
}

// ETag management codeunit
codeunit 50300 "ETag Manager"
{
    // Generate and store ETag for record
    procedure GenerateETag(TableID: Integer; RecordSystemId: Guid): Text
    var
        VersionControl: Record "Record Version Control";
        ETagValue: Text;
        VersionNumber: Integer;
    begin
        // Get or create version record
        if VersionControl.Get(TableID, RecordSystemId) then begin
            VersionNumber := VersionControl."Version Number" + 1;
        end else begin
            VersionControl.Init();
            VersionControl."Table ID" := TableID;
            VersionControl."Record SystemId" := RecordSystemId;
            VersionNumber := 1;
        end;
        
        // Generate ETag from version and timestamp
        ETagValue := StrSubstNo('%1-%2-%3', 
                                TableID, 
                                VersionNumber, 
                                Format(CurrentDateTime, 0, 9));
        
        // Update version control record
        VersionControl."Version Number" := VersionNumber;
        VersionControl."Last Modified DateTime" := CurrentDateTime;
        VersionControl."Modified By User" := UserId;
        VersionControl."ETag Value" := ETagValue;
        
        if VersionControl."Version Number" = 1 then
            VersionControl.Insert()
        else
            VersionControl.Modify();
            
        exit(ETagValue);
    end;
    
    // Validate ETag for concurrency control
    procedure ValidateETag(TableID: Integer; RecordSystemId: Guid; ProvidedETag: Text): Boolean
    var
        VersionControl: Record "Record Version Control";
    begin
        if not VersionControl.Get(TableID, RecordSystemId) then
            exit(false);  // No version record exists
            
        exit(VersionControl."ETag Value" = ProvidedETag);
    end;
    
    // Get current ETag for record
    procedure GetCurrentETag(TableID: Integer; RecordSystemId: Guid): Text
    var
        VersionControl: Record "Record Version Control";
    begin
        if VersionControl.Get(TableID, RecordSystemId) then
            exit(VersionControl."ETag Value");
        exit('');
    end;
}

// API page with advanced ETag implementation
page 50301 "Item Advanced ETag API"
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
                }
                field(displayName; Rec.Description)
                {
                    Caption = 'Display Name';
                }
                field(unitPrice; Rec."Unit Price")
                {
                    Caption = 'Unit Price';
                }
                field(inventory; Rec.Inventory)
                {
                    Caption = 'Inventory';
                    Editable = false;
                }
                field(etag; ETagValue)
                {
                    Caption = 'ETag';
                    Editable = false;
                }
                field(version; VersionNumber)
                {
                    Caption = 'Version';
                    Editable = false;
                }
            }
        }
    }
    
    var
        ETagValue: Text;
        VersionNumber: Integer;
        ETagManager: Codeunit "ETag Manager";
        ProvidedETag: Text;
    
    trigger OnAfterGetRecord()
    var
        VersionControl: Record "Record Version Control";
    begin
        // Get current ETag and version
        ETagValue := ETagManager.GetCurrentETag(Database::Item, Rec.SystemId);
        
        if VersionControl.Get(Database::Item, Rec.SystemId) then
            VersionNumber := VersionControl."Version Number"
        else
            VersionNumber := 0;
            
        // Generate ETag if not exists
        if ETagValue = '' then
            ETagValue := ETagManager.GenerateETag(Database::Item, Rec.SystemId);
    end;
    
    trigger OnInsertRecord(BelowxRec: Boolean): Boolean
    begin
        // Generate initial ETag for new record
        ETagValue := ETagManager.GenerateETag(Database::Item, Rec.SystemId);
        exit(true);
    end;
    
    trigger OnModifyRecord(): Boolean
    begin
        // Validate provided ETag before modification
        if ProvidedETag <> '' then begin
            if not ETagManager.ValidateETag(Database::Item, Rec.SystemId, ProvidedETag) then
                Error('Concurrency conflict detected. The record has been modified by another user. ETag: %1', ProvidedETag);
        end;
        
        // Generate new ETag after successful modification
        ETagValue := ETagManager.GenerateETag(Database::Item, Rec.SystemId);
        exit(true);
    end;
    
    trigger OnDeleteRecord(): Boolean
    var
        VersionControl: Record "Record Version Control";
    begin
        // Validate ETag before deletion
        if ProvidedETag <> '' then begin
            if not ETagManager.ValidateETag(Database::Item, Rec.SystemId, ProvidedETag) then
                Error('Concurrency conflict detected. Cannot delete record that has been modified.');
        end;
        
        // Clean up version control record
        if VersionControl.Get(Database::Item, Rec.SystemId) then
            VersionControl.Delete();
            
        exit(true);
    end;
}
```

## HTTP Header ETag Processing

```al
// Codeunit for processing ETag HTTP headers
codeunit 50301 "HTTP ETag Processor"
{
    // Extract ETag from If-Match header
    procedure ExtractETagFromHeader(HttpHeaders: Dictionary of [Text, Text]): Text
    var
        ETagHeader: Text;
        ETagValue: Text;
    begin
        // Check If-Match header for ETag value
        if HttpHeaders.Get('If-Match', ETagHeader) then begin
            // Remove quotes from ETag value
            ETagValue := ETagHeader.Replace('"', '');
            exit(ETagValue);
        end;
        
        // Check If-None-Match header
        if HttpHeaders.Get('If-None-Match', ETagHeader) then begin
            ETagValue := ETagHeader.Replace('"', '');
            exit(ETagValue);
        end;
        
        exit('');
    end;
    
    // Set ETag in response headers
    procedure SetETagResponseHeader(var HttpHeaders: Dictionary of [Text, Text]; ETagValue: Text)
    begin
        if ETagValue <> '' then
            HttpHeaders.Set('ETag', '"' + ETagValue + '"');
    end;
    
    // Process conditional request based on ETag
    procedure ProcessConditionalRequest(ProvidedETag: Text; CurrentETag: Text; RequestMethod: Text): Boolean
    begin
        case RequestMethod of
            'GET':
                // If-None-Match: return 304 if ETags match
                exit(ProvidedETag <> CurrentETag);
            'PUT', 'PATCH', 'DELETE':
                // If-Match: proceed only if ETags match
                exit(ProvidedETag = CurrentETag);
            else
                exit(true);
        end;
    end;
}

// API page with HTTP ETag header processing
page 50302 "Sales Order ETag API"
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
                field(totalAmount; TotalAmount)
                {
                    Caption = 'Total Amount';
                    Editable = false;
                }
                field(etag; ETagValue)
                {
                    Caption = 'ETag';
                    Editable = false;
                }
            }
        }
    }
    
    var
        ETagValue: Text;
        TotalAmount: Decimal;
        ETagManager: Codeunit "ETag Manager";
        HttpETagProcessor: Codeunit "HTTP ETag Processor";
        RequestETag: Text;
    
    trigger OnAfterGetRecord()
    begin
        // Calculate totals
        Rec.CalcFields("Amount Including VAT");
        TotalAmount := Rec."Amount Including VAT";
        
        // Generate ETag based on header and line modifications
        ETagValue := GenerateOrderETag();
    end;
    
    trigger OnModifyRecord(): Boolean
    var
        CurrentETag: Text;
    begin
        // Get current ETag for comparison
        CurrentETag := GenerateOrderETag();
        
        // Validate provided ETag matches current state
        if (RequestETag <> '') and (RequestETag <> CurrentETag) then
            Error('Concurrency conflict: Order has been modified by another user. Current ETag: %1, Provided ETag: %2', CurrentETag, RequestETag);
            
        exit(true);
    end;
    
    local procedure GenerateOrderETag(): Text
    var
        SalesLine: Record "Sales Line";
        ETagSource: Text;
        TypeHelper: Codeunit "Type Helper";
        LineModificationTime: DateTime;
    begin
        // Build ETag from header and all related lines
        ETagSource := Format(Rec.SystemModifiedAt, 0, 9);
        
        // Include line modifications in ETag
        SalesLine.SetRange("Document Type", Rec."Document Type");
        SalesLine.SetRange("Document No.", Rec."No.");
        if SalesLine.FindLast() then
            LineModificationTime := SalesLine.SystemModifiedAt
        else
            LineModificationTime := Rec.SystemModifiedAt;
            
        ETagSource += Format(LineModificationTime, 0, 9);
        ETagSource += Format(Rec.SystemId);
        
        // Generate hash-based ETag
        exit(TypeHelper.GetMD5Hash(ETagSource));
    end;
}
```

## Implementation Notes

**ETag Implementation Best Practices:**

1. **ETag Generation Strategies:**
   - Use SystemModifiedAt for simple timestamp-based ETags
   - Combine multiple factors (timestamp + SystemId + version) for robust ETags
   - Include related record modifications for composite entities
   - Use consistent hashing algorithms (MD5, SHA-256) for ETag values

2. **Concurrency Control:**
   - Validate ETags on all modification operations (PUT, PATCH, DELETE)
   - Return appropriate HTTP status codes (409 Conflict, 412 Precondition Failed)
   - Implement version tracking for detailed conflict resolution
   - Provide clear error messages for concurrency conflicts

3. **Performance Optimization:**
   - Index ETag fields for fast lookups
   - Cache ETag values to avoid recalculation
   - Use efficient ETag generation algorithms
   - Minimize database queries during ETag validation

4. **HTTP Integration:**
   - Process If-Match and If-None-Match headers correctly
   - Set ETag response headers for all API responses
   - Support conditional requests (304 Not Modified responses)
   - Handle weak vs strong ETag semantics appropriately