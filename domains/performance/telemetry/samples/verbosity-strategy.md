# Telemetry Verbosity Level Usage Examples - AL Code Samples

## Verbosity Strategy Implementation
```al
codeunit 50120 "Verbosity Strategy Examples"
{
    var
        CurrentVerbosityLevel: Verbosity;
        VerbosityConfig: Dictionary of [Text, Verbosity];

    // Example 1: Context-aware verbosity selection
    procedure LogWithContextualVerbosity(EventCategory: Text; Message: Text; CustomDimensions: Dictionary of [Text, Text])
    var
        SelectedVerbosity: Verbosity;
    begin
        SelectedVerbosity := DetermineVerbosityLevel(EventCategory);
        
        Session.LogMessage(GetNextEventId(), Message, SelectedVerbosity,
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    local procedure DetermineVerbosityLevel(EventCategory: Text): Verbosity
    begin
        // Production verbosity rules
        case EventCategory of
            'Error', 'Critical':
                exit(Verbosity::Error);
            'Warning', 'Performance':
                exit(Verbosity::Warning);
            'Info', 'Business':
                exit(Verbosity::Normal);
            'Debug', 'Trace', 'Development':
                if IsDebugMode() then
                    exit(Verbosity::Verbose)
                else
                    exit(Verbosity::Normal);
            else
                exit(Verbosity::Normal);
        end;
    end;

    // Example 2: Environment-based verbosity control
    procedure InitializeVerbosityStrategy()
    begin
        if IsProductionEnvironment() then
            InitializeProductionVerbosity()
        else if IsStagingEnvironment() then
            InitializeStagingVerbosity()
        else
            InitializeDevelopmentVerbosity();
    end;

    local procedure InitializeProductionVerbosity()
    begin
        // Production: Minimize noise, focus on actionable events
        VerbosityConfig.Add('UserActions', Verbosity::Normal);
        VerbosityConfig.Add('BusinessProcesses', Verbosity::Normal);
        VerbosityConfig.Add('Errors', Verbosity::Error);
        VerbosityConfig.Add('Performance', Verbosity::Warning);
        VerbosityConfig.Add('Debugging', Verbosity::Critical); // Only critical debug info
        VerbosityConfig.Add('Integration', Verbosity::Normal);
    end;

    local procedure InitializeDevelopmentVerbosity()
    begin
        // Development: Detailed logging for debugging
        VerbosityConfig.Add('UserActions', Verbosity::Verbose);
        VerbosityConfig.Add('BusinessProcesses', Verbosity::Verbose);
        VerbosityConfig.Add('Errors', Verbosity::Error);
        VerbosityConfig.Add('Performance', Verbosity::Verbose);
        VerbosityConfig.Add('Debugging', Verbosity::Verbose);
        VerbosityConfig.Add('Integration', Verbosity::Verbose);
    end;

    local procedure InitializeStagingVerbosity()
    begin
        // Staging: Balanced approach for pre-production testing
        VerbosityConfig.Add('UserActions', Verbosity::Normal);
        VerbosityConfig.Add('BusinessProcesses', Verbosity::Normal);
        VerbosityConfig.Add('Errors', Verbosity::Error);
        VerbosityConfig.Add('Performance', Verbosity::Normal);
        VerbosityConfig.Add('Debugging', Verbosity::Warning);
        VerbosityConfig.Add('Integration', Verbosity::Normal);
    end;

    // Helper methods
    local procedure IsDebugMode(): Boolean
    begin
        exit(false); // Implement actual debug mode detection
    end;

    local procedure IsProductionEnvironment(): Boolean
    begin
        exit(true); // Implement actual environment detection
    end;

    local procedure IsStagingEnvironment(): Boolean
    begin
        exit(false); // Implement actual staging detection
    end;

    local procedure GetNextEventId(): Text
    begin
        exit(Format(Random(9999), 4, '<Integer,4><Filler Character,0>'));
    end;
}
```

## Performance-Based Verbosity Control
```al
codeunit 50121 "Performance Verbosity Control"
{
    // Example 3: Performance threshold-based verbosity
    procedure LogPerformanceEvent(OperationName: Text; DurationMs: Integer; Context: Dictionary of [Text, Text])
    var
        CustomDimensions: Dictionary of [Text, Text];
        PerformanceVerbosity: Verbosity;
        Message: Text;
    begin
        CustomDimensions := Context;
        CustomDimensions.Add('OperationName', OperationName);
        CustomDimensions.Add('DurationMs', Format(DurationMs));
        
        PerformanceVerbosity := GetPerformanceVerbosity(OperationName, DurationMs);
        Message := StrSubstNo('Performance: %1 completed in %2ms', OperationName, DurationMs);
        
        Session.LogMessage('PERF' + Format(Random(999), 3, '<Integer,3><Filler Character,0>'), Message, PerformanceVerbosity,
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    local procedure GetPerformanceVerbosity(OperationName: Text; DurationMs: Integer): Verbosity
    var
        WarningThreshold: Integer;
        ErrorThreshold: Integer;
    begin
        GetPerformanceThresholds(OperationName, WarningThreshold, ErrorThreshold);
        
        case true of
            DurationMs >= ErrorThreshold:
                exit(Verbosity::Error); // Critical performance issue
            DurationMs >= WarningThreshold:
                exit(Verbosity::Warning); // Performance concern
            DurationMs >= (WarningThreshold div 2):
                exit(Verbosity::Normal); // Notable but acceptable
            else
                exit(Verbosity::Verbose); // Fast operations - detailed tracking only
        end;
    end;

    local procedure GetPerformanceThresholds(OperationName: Text; var WarningThreshold: Integer; var ErrorThreshold: Integer)
    begin
        // Operation-specific thresholds
        case OperationName of
            'DatabaseQuery':
                begin
                    WarningThreshold := 1000; // 1 second
                    ErrorThreshold := 5000; // 5 seconds
                end;
            'ReportGeneration':
                begin
                    WarningThreshold := 10000; // 10 seconds
                    ErrorThreshold := 30000; // 30 seconds
                end;
            'APICall':
                begin
                    WarningThreshold := 2000; // 2 seconds
                    ErrorThreshold := 10000; // 10 seconds
                end;
            else
                begin
                    WarningThreshold := 5000; // Default 5 seconds
                    ErrorThreshold := 15000; // Default 15 seconds
                end;
        end;
    end;
}
```

## User Role-Based Verbosity
```al
codeunit 50122 "Role Based Verbosity"
{
    // Example 4: User role-based verbosity control
    procedure LogUserAction(ActionType: Text; UserID: Code[50]; Context: Text)
    var
        CustomDimensions: Dictionary of [Text, Text];
        UserVerbosity: Verbosity;
    begin
        CustomDimensions.Add('UserID', UserID);
        CustomDimensions.Add('ActionType', ActionType);
        CustomDimensions.Add('Context', Context);
        
        UserVerbosity := GetUserActionVerbosity(UserID, ActionType);
        
        Session.LogMessage('USER' + Format(Random(999), 3, '<Integer,3><Filler Character,0>'), 
            StrSubstNo('User action: %1', ActionType), UserVerbosity,
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    local procedure GetUserActionVerbosity(UserID: Code[50]; ActionType: Text): Verbosity
    begin
        // Higher verbosity for administrative users in development
        if IsAdministrativeUser(UserID) and not IsProductionEnvironment() then
            exit(Verbosity::Verbose);
            
        case ActionType of
            'Login', 'Logout':
                exit(Verbosity::Normal);
            'DataExport', 'ReportGeneration':
                exit(Verbosity::Normal);
            'ConfigurationChange':
                exit(Verbosity::Warning); // Important to track
            'SecurityElevation':
                exit(Verbosity::Critical); // Always log security events
            else
                exit(Verbosity::Normal);
        end;
    end;

    local procedure IsAdministrativeUser(UserID: Code[50]): Boolean
    var
        AccessControl: Record "Access Control";
    begin
        // Check if user has administrative permissions
        AccessControl.SetRange("User Security ID", UserID);
        AccessControl.SetRange("Role ID", 'SUPER');
        exit(not AccessControl.IsEmpty);
    end;

    local procedure IsProductionEnvironment(): Boolean
    begin
        // Implement environment detection logic
        exit(true);
    end;
}
```