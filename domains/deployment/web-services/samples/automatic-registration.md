# API Page Automatic Web Service Registration - AL Code Sample

## Basic Automatic Registration

```al
page 50400 "Auto-Register Customer API"
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
    
    // BC automatically registers this as web service based on API properties
    // Web service name: contoso_customers_v1.0
    // URL: /api/contoso/customers/v1.0/companies({company-id})/customers
    
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
            }
        }
    }
}

// Verification codeunit to check automatic registration
codeunit 50400 "API Registration Checker"
{
    procedure VerifyAPIRegistration(APIPublisher: Text; APIGroup: Text; APIVersion: Text): Boolean
    var
        WebService: Record "Web Service";
        ExpectedServiceName: Text;
    begin
        // Build expected web service name
        ExpectedServiceName := StrSubstNo('%1_%2_%3', APIPublisher, APIGroup, APIVersion);
        
        // Check if web service exists
        WebService.SetRange("Service Name", ExpectedServiceName);
        WebService.SetRange(Published, true);
        
        if WebService.FindFirst() then begin
            Message('API automatically registered as: %1', WebService."Service Name");
            exit(true);
        end else begin
            Message('API registration not found for: %1', ExpectedServiceName);
            exit(false);
        end;
    end;
    
    procedure ListAllAPIServices()
    var
        WebService: Record "Web Service";
        ServiceList: Text;
    begin
        // Find all API-type web services
        WebService.SetRange("Object Type", WebService."Object Type"::Page);
        WebService.SetRange(Published, true);
        WebService.SetFilter("Service Name", '*_*_v*');  // Pattern for API services
        
        if WebService.FindSet() then begin
            ServiceList := 'Registered API Services:\';
            repeat
                ServiceList += StrSubstNo('- %1 (Object ID: %2)\', 
                                        WebService."Service Name", 
                                        WebService."Object ID");
            until WebService.Next() = 0;
            
            Message(ServiceList);
        end else
            Message('No API services found');
    end;
}
```

## Registration with Custom Settings

```al
page 50401 "Custom Settings Item API"
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
    
    // Custom web service registration during install
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
                field(description; Rec.Description)
                {
                    Caption = 'Description';
                }
                field(unitPrice; Rec."Unit Price")
                {
                    Caption = 'Unit Price';
                }
            }
        }
    }
}

// Install codeunit for custom web service registration
codeunit 50401 "API Service Registration"
{
    Subtype = Install;
    
    trigger OnInstallAppPerCompany()
    begin
        RegisterAPIServices();
    end;
    
    procedure RegisterAPIServices()
    var
        WebService: Record "Web Service";
        WebServiceManagement: Codeunit "Web Service Management";
    begin
        // Register Item API with custom settings
        if not WebService.Get(WebService."Object Type"::Page, 50401) then begin
            WebService.Init();
            WebService."Object Type" := WebService."Object Type"::Page;
            WebService."Object ID" := 50401;
            WebService."Service Name" := 'contoso_inventory_v1.0';
            WebService.Published := true;
            WebService.Insert(true);
            
            // Configure additional OData settings
            ConfigureODataSettings(WebService);
        end;
        
        // Register additional related services
        RegisterRelatedServices();
    end;
    
    local procedure ConfigureODataSettings(var WebService: Record "Web Service")
    var
        TenantWebService: Record "Tenant Web Service";
    begin
        // Configure tenant-specific OData settings
        if not TenantWebService.Get(WebService."Object Type", WebService."Service Name") then begin
            TenantWebService.Init();
            TenantWebService."Object Type" := WebService."Object Type";
            TenantWebService."Service Name" := WebService."Service Name";
            TenantWebService."Object ID" := WebService."Object ID";
            TenantWebService.Published := true;
            TenantWebService.Insert(true);
        end;
    end;
    
    local procedure RegisterRelatedServices()
    var
        WebService: Record "Web Service";
    begin
        // Register related codeunit services for API support
        if not WebService.Get(WebService."Object Type"::Codeunit, 50401) then begin
            WebService.Init();
            WebService."Object Type" := WebService."Object Type"::Codeunit;
            WebService."Object ID" := 50401;
            WebService."Service Name" := 'APIServiceRegistration';
            WebService.Published := true;
            WebService.Insert(true);
        end;
    end;
}

// Upgrade codeunit for handling service registration updates
codeunit 50402 "API Service Upgrade"
{
    Subtype = Upgrade;
    
    trigger OnUpgradePerCompany()
    begin
        UpdateAPIServiceRegistrations();
    end;
    
    procedure UpdateAPIServiceRegistrations()
    var
        WebService: Record "Web Service";
        APIRegistration: Codeunit "API Service Registration";
    begin
        // Update existing API service registrations
        WebService.SetRange("Object Type", WebService."Object Type"::Page);
        WebService.SetFilter("Service Name", 'contoso_*');
        
        if WebService.FindSet() then
            repeat
                // Ensure all API services are published
                if not WebService.Published then begin
                    WebService.Published := true;
                    WebService.Modify();
                end;
            until WebService.Next() = 0;
            
        // Register any new services
        APIRegistration.RegisterAPIServices();
    end;
}
```

## Registration Monitoring and Management

```al
// Management codeunit for API service oversight
codeunit 50403 "API Service Manager"
{
    // Monitor API service status and performance
    procedure MonitorAPIServices()
    var
        WebService: Record "Web Service";
        TenantWebService: Record "Tenant Web Service";
        UnpublishedCount: Integer;
        TotalCount: Integer;
    begin
        // Count API services
        WebService.SetRange("Object Type", WebService."Object Type"::Page);
        WebService.SetFilter("Service Name", '*_*_v*');  // API naming pattern
        TotalCount := WebService.Count();
        
        WebService.SetRange(Published, false);
        UnpublishedCount := WebService.Count();
        
        // Report status
        Message('API Service Status:\Total Services: %1\Published: %2\Unpublished: %3', 
               TotalCount, 
               TotalCount - UnpublishedCount, 
               UnpublishedCount);
               
        // Check for registration issues
        CheckRegistrationIssues();
    end;
    
    local procedure CheckRegistrationIssues()
    var
        WebService: Record "Web Service";
        TenantWebService: Record "Tenant Web Service";
        IssueList: Text;
    begin
        // Check for services without tenant registration
        WebService.SetRange("Object Type", WebService."Object Type"::Page);
        WebService.SetRange(Published, true);
        WebService.SetFilter("Service Name", '*_*_v*');
        
        if WebService.FindSet() then
            repeat
                if not TenantWebService.Get(WebService."Object Type", WebService."Service Name") then
                    IssueList += StrSubstNo('- %1 missing tenant registration\', WebService."Service Name");
            until WebService.Next() = 0;
            
        if IssueList <> '' then
            Message('Registration Issues Found:\%1', IssueList);
    end;
    
    // Bulk operations for API service management
    procedure PublishAllAPIServices()
    var
        WebService: Record "Web Service";
        PublishedCount: Integer;
    begin
        WebService.SetRange("Object Type", WebService."Object Type"::Page);
        WebService.SetFilter("Service Name", '*_*_v*');
        WebService.SetRange(Published, false);
        
        if WebService.FindSet() then
            repeat
                WebService.Published := true;
                WebService.Modify();
                PublishedCount += 1;
            until WebService.Next() = 0;
            
        Message('Published %1 API services', PublishedCount);
    end;
    
    procedure UnpublishAPIServices(ServicePattern: Text)
    var
        WebService: Record "Web Service";
        UnpublishedCount: Integer;
    begin
        WebService.SetRange("Object Type", WebService."Object Type"::Page);
        WebService.SetFilter("Service Name", ServicePattern);
        WebService.SetRange(Published, true);
        
        if WebService.FindSet() then
            repeat
                WebService.Published := false;
                WebService.Modify();
                UnpublishedCount += 1;
            until WebService.Next() = 0;
            
        Message('Unpublished %1 API services matching pattern: %2', UnpublishedCount, ServicePattern);
    end;
}

// Page for API service administration
page 50403 "API Service Administration"
{
    PageType = List;
    ApplicationArea = All;
    UsageCategory = Administration;
    SourceTable = "Web Service";
    SourceTableView = WHERE("Object Type" = CONST(Page));
    Caption = 'API Service Administration';
    
    layout
    {
        area(Content)
        {
            repeater(GroupName)
            {
                field("Service Name"; Rec."Service Name")
                {
                    ApplicationArea = All;
                    Caption = 'Service Name';
                }
                field("Object ID"; Rec."Object ID")
                {
                    ApplicationArea = All;
                    Caption = 'Page ID';
                }
                field(Published; Rec.Published)
                {
                    ApplicationArea = All;
                    Caption = 'Published';
                }
                field("URL"; ServiceURL)
                {
                    ApplicationArea = All;
                    Caption = 'Service URL';
                    Editable = false;
                }
            }
        }
    }
    
    actions
    {
        area(Processing)
        {
            action(PublishService)
            {
                ApplicationArea = All;
                Caption = 'Publish Service';
                Image = Process;
                
                trigger OnAction()
                begin
                    Rec.Published := true;
                    Rec.Modify();
                    Message('Service %1 published successfully', Rec."Service Name");
                end;
            }
            
            action(UnpublishService)
            {
                ApplicationArea = All;
                Caption = 'Unpublish Service';
                Image = Stop;
                
                trigger OnAction()
                begin
                    Rec.Published := false;
                    Rec.Modify();
                    Message('Service %1 unpublished successfully', Rec."Service Name");
                end;
            }
            
            action(TestService)
            {
                ApplicationArea = All;
                Caption = 'Test Service';
                Image = TestFile;
                
                trigger OnAction()
                var
                    APIServiceManager: Codeunit "API Service Manager";
                begin
                    // Test API service accessibility
                    TestAPIService();
                end;
            }
        }
    }
    
    var
        ServiceURL: Text;
    
    trigger OnAfterGetRecord()
    begin
        // Build service URL for display
        if Rec.Published then
            ServiceURL := BuildAPIUrl(Rec."Service Name")
        else
            ServiceURL := 'Service not published';
    end;
    
    local procedure BuildAPIUrl(ServiceName: Text): Text
    var
        BaseUrl: Text;
        CompanyName: Text;
    begin
        BaseUrl := GetUrl(ClientType::ODataV4, CompanyName(), ObjectType::Page, 0);
        exit(StrSubstNo('%1/%2', BaseUrl, ServiceName));
    end;
    
    local procedure TestAPIService()
    var
        HttpClient: HttpClient;
        RequestMessage: HttpRequestMessage;
        ResponseMessage: HttpResponseMessage;
        TestUrl: Text;
    begin
        TestUrl := BuildAPIUrl(Rec."Service Name");
        
        RequestMessage.Method := 'GET';
        RequestMessage.SetRequestUri(TestUrl);
        
        if HttpClient.Send(RequestMessage, ResponseMessage) then begin
            if ResponseMessage.IsSuccessStatusCode() then
                Message('API service test successful. Status: %1', ResponseMessage.HttpStatusCode())
            else
                Message('API service test failed. Status: %1', ResponseMessage.HttpStatusCode());
        end else
            Message('Failed to connect to API service');
    end;
}
```

## Implementation Notes

**Automatic Registration Best Practices:**

1. **Service Naming Conventions:**
   - Follow consistent naming patterns: {publisher}_{group}_{version}
   - Use lowercase and underscores for compatibility
   - Include version information for API evolution
   - Avoid special characters that might cause URL issues

2. **Registration Management:**
   - Use install/upgrade codeunits for automatic registration
   - Implement monitoring for registration health checks
   - Provide administrative pages for manual management
   - Include tenant-specific configuration when needed

3. **Deployment Considerations:**
   - Register services during app installation
   - Handle registration updates during app upgrades
   - Provide rollback capabilities for failed registrations
   - Test service accessibility after registration

4. **Security and Access Control:**
   - Configure appropriate permissions for service objects
   - Implement authentication requirements for published services
   - Monitor service usage and access patterns
   - Provide mechanisms to unpublish services when needed