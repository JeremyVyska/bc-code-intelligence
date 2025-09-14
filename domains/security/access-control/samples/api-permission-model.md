# API Page Permission Model - AL Code Sample

## Basic Permission Set for API Access

```al
permissionset 50100 "Customer API Access"
{
    Caption = 'Customer API Access';
    Access = Public;
    Assignable = true;
    
    Permissions = 
        tabledata Customer = RI,           // Read and Insert permissions
        tabledata "Customer Bank Account" = R,  // Read-only access to related data
        page "Customer API" = X,           // Execute permission for API page
        codeunit "Customer Management" = X; // Execute related business logic
}

// Example API page with permission requirements
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
            }
        }
    }
}
```

## Advanced Permission Model with Role-Based Access

```al
// Base API permission set - minimal access
permissionset 50101 "Base API Reader"
{
    Caption = 'Base API Reader';
    Access = Public;
    Assignable = false;  // Only used as building block
    
    Permissions = 
        tabledata Customer = R,
        tabledata Item = R,
        tabledata Vendor = R;
}

// Extended permission set for API writers
permissionset 50102 "API Data Writer"
{
    Caption = 'API Data Writer';
    Access = Public;
    Assignable = true;
    
    IncludedPermissionSets = "Base API Reader";  // Inherit read permissions
    
    Permissions = 
        tabledata Customer = RIM,          // Read, Insert, Modify
        tabledata Item = RIM,
        tabledata Vendor = RIM,
        tabledata "Gen. Journal Line" = RIMD,  // Full access for transaction APIs
        page "Customer API" = X,
        page "Item API" = X,
        page "Vendor API" = X,
        codeunit "API Helper Functions" = X;
}

// Administrative permission set with full API control
permissionset 50103 "API Administrator"
{
    Caption = 'API Administrator';
    Access = Public;
    Assignable = true;
    
    IncludedPermissionSets = "API Data Writer";
    
    Permissions = 
        tabledata "Web Service" = RIMD,    // Manage web service registrations
        tabledata "OAuth 2.0 Setup" = RIMD,   // Manage API authentication
        tabledata "User Setup" = RM,      // Modify user API permissions
        page "Web Services" = X,
        codeunit "Web Service Management" = X;
}
```

## Permission Validation in API Implementation

```al
codeunit 50100 "API Security Helper"
{
    // Validate user has required permissions for API operation
    procedure ValidateAPIPermissions(TableID: Integer; Operation: Text): Boolean
    var
        Permission: Record Permission;
        UserPermissions: Record "Access Control";
    begin
        // Check if user has appropriate table permissions
        case Operation of
            'READ':
                exit(HasTablePermission(TableID, 'R'));
            'CREATE':
                exit(HasTablePermission(TableID, 'I'));
            'UPDATE':
                exit(HasTablePermission(TableID, 'M'));
            'DELETE':
                exit(HasTablePermission(TableID, 'D'));
            else
                exit(false);
        end;
    end;
    
    local procedure HasTablePermission(TableID: Integer; PermissionType: Text): Boolean
    var
        Permission: Record Permission;
        AccessControl: Record "Access Control";
    begin
        AccessControl.SetRange("User Security ID", UserSecurityId());
        if AccessControl.FindSet() then
            repeat
                Permission.SetRange("Role ID", AccessControl."Role ID");
                Permission.SetRange("Object Type", Permission."Object Type"::"Table Data");
                Permission.SetRange("Object ID", TableID);
                if Permission.FindFirst() then
                    if StrPos(Permission."Read Permission", PermissionType) > 0 then
                        exit(true);
            until AccessControl.Next() = 0;
        exit(false);
    end;
}

// API page with permission enforcement
page 50101 "Secure Customer API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'customers';
    APIVersion = 'v1.0';
    EntityName = 'secureCustomer';
    EntitySetName = 'secureCustomers';
    SourceTable = Customer;
    DelayedInsert = true;
    ODataKeyFields = SystemId;
    
    trigger OnOpenPage()
    var
        APISecurityHelper: Codeunit "API Security Helper";
    begin
        // Validate user has read permissions
        if not APISecurityHelper.ValidateAPIPermissions(Database::Customer, 'READ') then
            Error('Insufficient permissions to access Customer API');
    end;
    
    trigger OnInsertRecord(BelowxRec: Boolean): Boolean
    var
        APISecurityHelper: Codeunit "API Security Helper";
    begin
        // Validate user has insert permissions
        if not APISecurityHelper.ValidateAPIPermissions(Database::Customer, 'CREATE') then
            Error('Insufficient permissions to create customers via API');
        exit(true);
    end;
    
    trigger OnModifyRecord(): Boolean
    var
        APISecurityHelper: Codeunit "API Security Helper";
    begin
        // Validate user has modify permissions
        if not APISecurityHelper.ValidateAPIPermissions(Database::Customer, 'UPDATE') then
            Error('Insufficient permissions to modify customers via API');
        exit(true);
    end;
}
```

## OAuth 2.0 Integration with Permissions

```al
// OAuth setup with permission scope mapping
table 50100 "API OAuth Configuration"
{
    Caption = 'API OAuth Configuration';
    DataClassification = SystemMetadata;
    
    fields
    {
        field(1; "Scope Name"; Text[100])
        {
            Caption = 'Scope Name';
        }
        field(2; "Permission Set Code"; Code[20])
        {
            Caption = 'Permission Set Code';
            TableRelation = "Permission Set"."Role ID";
        }
        field(3; "Description"; Text[250])
        {
            Caption = 'Description';
        }
        field(4; "Active"; Boolean)
        {
            Caption = 'Active';
            InitValue = true;
        }
    }
    
    keys
    {
        key(PK; "Scope Name")
        {
            Clustered = true;
        }
    }
}

// Codeunit for OAuth scope to permission mapping
codeunit 50101 "OAuth Permission Manager"
{
    // Map OAuth scopes to BC permission sets
    procedure AssignPermissionsFromOAuthScope(UserID: Guid; OAuthScope: Text)
    var
        OAuthConfig: Record "API OAuth Configuration";
        AccessControl: Record "Access Control";
        Scopes: List of [Text];
        Scope: Text;
    begin
        // Parse OAuth scopes (space-separated)
        Scopes := OAuthScope.Split(' ');
        
        foreach Scope in Scopes do begin
            OAuthConfig.Reset();
            OAuthConfig.SetRange("Scope Name", Scope);
            OAuthConfig.SetRange(Active, true);
            if OAuthConfig.FindSet() then
                repeat
                    // Assign corresponding permission set
                    if not AccessControl.Get(UserID, OAuthConfig."Permission Set Code", '', '') then begin
                        AccessControl.Init();
                        AccessControl."User Security ID" := UserID;
                        AccessControl."Role ID" := OAuthConfig."Permission Set Code";
                        AccessControl.Insert();
                    end;
                until OAuthConfig.Next() = 0;
        end;
    end;
    
    // Validate OAuth token has required scope for operation
    procedure ValidateOAuthScope(RequiredScope: Text; ProvidedScopes: Text): Boolean
    var
        Scopes: List of [Text];
        Scope: Text;
    begin
        Scopes := ProvidedScopes.Split(' ');
        foreach Scope in Scopes do
            if Scope = RequiredScope then
                exit(true);
        exit(false);
    end;
}
```

## Implementation Notes

**Permission Model Best Practices:**

1. **Principle of Least Privilege:**
   - Grant minimum permissions required for API functionality
   - Use specific table permissions rather than blanket access
   - Separate read and write permissions into different sets

2. **Role-Based Access Control:**
   - Create permission sets that align with business roles
   - Use inheritance to build complex permission structures
   - Implement permission validation at API endpoints

3. **OAuth Integration:**
   - Map OAuth scopes to specific BC permission sets
   - Validate scopes before processing API requests
   - Implement dynamic permission assignment based on token scopes

4. **Security Monitoring:**
   - Log permission failures for security auditing
   - Monitor API access patterns for unusual behavior
   - Implement rate limiting based on permission levels