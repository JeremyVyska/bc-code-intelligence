# Extension Telemetry Isolation Implementation - AL Code Samples

## Extension-Specific Telemetry Architecture
```al
codeunit 50150 "Extension Telemetry Manager"
{
    var
        ExtensionIdentifier: Text;
        ExtensionVersion: Text;
        ExtensionPublisher: Text;
        TelemetryPrefix: Text;

    trigger OnRun()
    begin
        InitializeExtensionTelemetry();
    end;

    // Example 1: Extension isolation initialization
    local procedure InitializeExtensionTelemetry()
    var
        ModuleInfo: ModuleInfo;
    begin
        NavApp.GetCurrentModuleInfo(ModuleInfo);
        ExtensionIdentifier := Format(ModuleInfo.Id);
        ExtensionVersion := Format(ModuleInfo.AppVersion);
        ExtensionPublisher := ModuleInfo.Publisher;
        
        // Create unique telemetry prefix for this extension
        TelemetryPrefix := 'EXT_' + ModuleInfo.Name.Replace(' ', '') + '_';
        
        LogExtensionInitialization();
    end;

    local procedure LogExtensionInitialization()
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions := BuildExtensionBaseDimensions();
        CustomDimensions.Add('EventType', 'ExtensionInitialization');
        CustomDimensions.Add('InitializationTime', Format(CurrentDateTime, 0, 9));
        
        Session.LogMessage(GetExtensionEventId('INIT001'), 
            StrSubstNo('Extension %1 telemetry initialized', ExtensionIdentifier),
            Verbosity::Normal, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 2: Isolated extension event logging
    procedure LogExtensionEvent(EventCategory: Text; EventName: Text; EventData: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        EventId: Text;
    begin
        CustomDimensions := BuildExtensionBaseDimensions();
        CustomDimensions.Add('EventCategory', EventCategory);
        CustomDimensions.Add('EventName', EventName);
        
        // Add extension-specific event data
        AddEventDataToDimensions(CustomDimensions, EventData);
        
        EventId := GetExtensionEventId(EventCategory + '_' + EventName);
        
        Session.LogMessage(EventId, StrSubstNo('Extension event: %1.%2', EventCategory, EventName),
            GetEventVerbosity(EventCategory), DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 3: Extension error isolation
    procedure LogExtensionError(ErrorContext: Text; ErrorMessage: Text; ErrorCode: Text)
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions := BuildExtensionBaseDimensions();
        CustomDimensions.Add('EventType', 'ExtensionError');
        CustomDimensions.Add('ErrorContext', ErrorContext);
        CustomDimensions.Add('ErrorMessage', ErrorMessage);
        CustomDimensions.Add('ErrorCode', ErrorCode);
        CustomDimensions.Add('CallStack', GetExtensionCallStack());
        
        Session.LogMessage(GetExtensionEventId('ERROR_' + ErrorCode),
            StrSubstNo('Extension error in %1: %2', ErrorContext, ErrorMessage),
            Verbosity::Error, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    local procedure BuildExtensionBaseDimensions(): Dictionary of [Text, Text]
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions.Add('ExtensionId', ExtensionIdentifier);
        CustomDimensions.Add('ExtensionVersion', ExtensionVersion);
        CustomDimensions.Add('ExtensionPublisher', ExtensionPublisher);
        CustomDimensions.Add('CompanyName', CompanyName());
        CustomDimensions.Add('UserID', UserId());
        CustomDimensions.Add('SessionId', Format(SessionId()));
        CustomDimensions.Add('TelemetryTimestamp', Format(CurrentDateTime, 0, 9));
        
        exit(CustomDimensions);
    end;

    local procedure GetExtensionEventId(EventSuffix: Text): Text
    begin
        exit(TelemetryPrefix + EventSuffix);
    end;

    local procedure GetEventVerbosity(EventCategory: Text): Verbosity
    begin
        case EventCategory of
            'Error': exit(Verbosity::Error);
            'Warning': exit(Verbosity::Warning);
            'Performance': exit(Verbosity::Normal);
            'Debug': exit(Verbosity::Verbose);
            else exit(Verbosity::Normal);
        end;
    end;

    local procedure AddEventDataToDimensions(var CustomDimensions: Dictionary of [Text, Text]; EventData: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in EventData.Keys do
            CustomDimensions.Add('EventData_' + Key, EventData.Get(Key));
    end;

    local procedure GetExtensionCallStack(): Text
    begin
        // Implement call stack capture for extension context
        exit('Extension call stack not available');
    end;
}
```

## Cross-Extension Interaction Tracking
```al
codeunit 50151 "Cross Extension Telemetry"
{
    // Example 4: Inter-extension communication tracking
    procedure LogExtensionInteraction(TargetExtension: Text; InteractionType: Text; InteractionData: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        SourceExtension: Text;
        ModuleInfo: ModuleInfo;
    begin
        NavApp.GetCurrentModuleInfo(ModuleInfo);
        SourceExtension := Format(ModuleInfo.Id);
        
        CustomDimensions.Add('SourceExtension', SourceExtension);
        CustomDimensions.Add('TargetExtension', TargetExtension);
        CustomDimensions.Add('InteractionType', InteractionType);
        CustomDimensions.Add('EventType', 'ExtensionInteraction');
        
        AddInteractionDataToDimensions(CustomDimensions, InteractionData);
        
        Session.LogMessage('EXTINT_' + InteractionType,
            StrSubstNo('Extension interaction: %1 -> %2 (%3)', SourceExtension, TargetExtension, InteractionType),
            Verbosity::Normal, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 5: Extension API usage tracking
    procedure LogExtensionAPIUsage(APIEndpoint: Text; APIOperation: Text; ClientExtension: Text; UsageMetrics: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        ProviderExtension: Text;
        ModuleInfo: ModuleInfo;
    begin
        NavApp.GetCurrentModuleInfo(ModuleInfo);
        ProviderExtension := Format(ModuleInfo.Id);
        
        CustomDimensions.Add('ProviderExtension', ProviderExtension);
        CustomDimensions.Add('ClientExtension', ClientExtension);
        CustomDimensions.Add('APIEndpoint', APIEndpoint);
        CustomDimensions.Add('APIOperation', APIOperation);
        CustomDimensions.Add('EventType', 'ExtensionAPIUsage');
        
        AddUsageMetricsToDimensions(CustomDimensions, UsageMetrics);
        
        Session.LogMessage('EXTAPI_' + APIOperation,
            StrSubstNo('Extension API usage: %1.%2 by %3', APIEndpoint, APIOperation, ClientExtension),
            Verbosity::Normal, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 6: Extension feature usage tracking
    procedure LogExtensionFeatureUsage(FeatureName: Text; UsageType: Text; FeatureData: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        ModuleInfo: ModuleInfo;
    begin
        NavApp.GetCurrentModuleInfo(ModuleInfo);
        
        CustomDimensions.Add('ExtensionId', Format(ModuleInfo.Id));
        CustomDimensions.Add('FeatureName', FeatureName);
        CustomDimensions.Add('UsageType', UsageType);
        CustomDimensions.Add('EventType', 'ExtensionFeatureUsage');
        CustomDimensions.Add('UserID', UserId());
        CustomDimensions.Add('CompanyName', CompanyName());
        
        AddFeatureDataToDimensions(CustomDimensions, FeatureData);
        
        Session.LogMessage('EXTFEAT_' + FeatureName,
            StrSubstNo('Extension feature usage: %1 (%2)', FeatureName, UsageType),
            Verbosity::Normal, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    local procedure AddInteractionDataToDimensions(var CustomDimensions: Dictionary of [Text, Text]; InteractionData: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in InteractionData.Keys do
            CustomDimensions.Add('Interaction_' + Key, InteractionData.Get(Key));
    end;

    local procedure AddUsageMetricsToDimensions(var CustomDimensions: Dictionary of [Text, Text]; UsageMetrics: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in UsageMetrics.Keys do
            CustomDimensions.Add('Metric_' + Key, UsageMetrics.Get(Key));
    end;

    local procedure AddFeatureDataToDimensions(var CustomDimensions: Dictionary of [Text, Text]; FeatureData: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in FeatureData.Keys do
            CustomDimensions.Add('Feature_' + Key, FeatureData.Get(Key));
    end;
}
```

## Extension Configuration and Lifecycle
```al
codeunit 50152 "Extension Lifecycle Telemetry"
{
    // Example 7: Extension lifecycle event tracking
    procedure LogExtensionLifecycleEvent(LifecycleEvent: Text; EventContext: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        ModuleInfo: ModuleInfo;
    begin
        NavApp.GetCurrentModuleInfo(ModuleInfo);
        
        CustomDimensions.Add('ExtensionId', Format(ModuleInfo.Id));
        CustomDimensions.Add('ExtensionName', ModuleInfo.Name);
        CustomDimensions.Add('ExtensionPublisher', ModuleInfo.Publisher);
        CustomDimensions.Add('ExtensionVersion', Format(ModuleInfo.AppVersion));
        CustomDimensions.Add('LifecycleEvent', LifecycleEvent);
        CustomDimensions.Add('EventType', 'ExtensionLifecycle');
        
        AddEventContextToDimensions(CustomDimensions, EventContext);
        
        Session.LogMessage('EXTLIFE_' + LifecycleEvent,
            StrSubstNo('Extension lifecycle event: %1 for %2', LifecycleEvent, ModuleInfo.Name),
            GetLifecycleVerbosity(LifecycleEvent), DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 8: Extension configuration tracking
    procedure LogExtensionConfiguration(ConfigurationArea: Text; ConfigurationChange: Text; ConfigurationData: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        ModuleInfo: ModuleInfo;
    begin
        NavApp.GetCurrentModuleInfo(ModuleInfo);
        
        CustomDimensions.Add('ExtensionId', Format(ModuleInfo.Id));
        CustomDimensions.Add('ConfigurationArea', ConfigurationArea);
        CustomDimensions.Add('ConfigurationChange', ConfigurationChange);
        CustomDimensions.Add('EventType', 'ExtensionConfiguration');
        CustomDimensions.Add('UserID', UserId());
        CustomDimensions.Add('ConfigurationTime', Format(CurrentDateTime, 0, 9));
        
        AddConfigurationDataToDimensions(CustomDimensions, ConfigurationData);
        
        Session.LogMessage('EXTCONF_' + ConfigurationArea,
            StrSubstNo('Extension configuration: %1 - %2', ConfigurationArea, ConfigurationChange),
            Verbosity::Normal, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    // Example 9: Multi-tenant extension isolation
    procedure LogTenantSpecificEvent(EventName: Text; TenantContext: Text; EventData: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        ModuleInfo: ModuleInfo;
    begin
        NavApp.GetCurrentModuleInfo(ModuleInfo);
        
        CustomDimensions.Add('ExtensionId', Format(ModuleInfo.Id));
        CustomDimensions.Add('TenantId', TenantId());
        CustomDimensions.Add('CompanyName', CompanyName());
        CustomDimensions.Add('TenantContext', TenantContext);
        CustomDimensions.Add('EventName', EventName);
        CustomDimensions.Add('EventType', 'TenantSpecificEvent');
        
        AddTenantEventDataToDimensions(CustomDimensions, EventData);
        
        Session.LogMessage('EXTTENANT_' + EventName,
            StrSubstNo('Tenant-specific extension event: %1 (Tenant: %2)', EventName, TenantId()),
            Verbosity::Normal, DataClassification::SystemMetadata,
            TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    local procedure GetLifecycleVerbosity(LifecycleEvent: Text): Verbosity
    begin
        case LifecycleEvent of
            'Install', 'Uninstall', 'Upgrade':
                exit(Verbosity::Critical); // Important lifecycle events
            'Configuration', 'FeatureEnabled':
                exit(Verbosity::Warning);
            'Startup', 'Shutdown':
                exit(Verbosity::Normal);
            else
                exit(Verbosity::Verbose);
        end;
    end;

    local procedure AddEventContextToDimensions(var CustomDimensions: Dictionary of [Text, Text]; EventContext: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in EventContext.Keys do
            CustomDimensions.Add('Context_' + Key, EventContext.Get(Key));
    end;

    local procedure AddConfigurationDataToDimensions(var CustomDimensions: Dictionary of [Text, Text]; ConfigurationData: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in ConfigurationData.Keys do
            CustomDimensions.Add('Config_' + Key, ConfigurationData.Get(Key));
    end;

    local procedure AddTenantEventDataToDimensions(var CustomDimensions: Dictionary of [Text, Text]; EventData: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in EventData.Keys do
            CustomDimensions.Add('TenantData_' + Key, EventData.Get(Key));
    end;
}
```