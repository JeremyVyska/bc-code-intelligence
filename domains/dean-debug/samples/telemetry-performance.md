# Telemetry Performance Optimization Patterns - AL Code Samples

## High-Performance Telemetry Architecture
```al
codeunit 50130 "Performance Telemetry Manager"
{
    var
        TelemetryBuffer: List of [Text];
        BatchSize: Integer;
        LastFlushTime: DateTime;
        FlushIntervalMs: Integer;
        IsBufferingEnabled: Boolean;

    // Example 1: Batched telemetry for high-volume scenarios
    trigger OnRun()
    begin
        InitializePerformanceSettings();
    end;

    local procedure InitializePerformanceSettings()
    begin
        BatchSize := 50;
        FlushIntervalMs := 30000; // 30 seconds
        IsBufferingEnabled := not IsDebugMode();
        
        if IsBufferingEnabled then
            TaskScheduler.CreateTask(Codeunit::"Performance Telemetry Manager", 0, true, CompanyName(),
                CurrentDateTime + FlushIntervalMs, RecordId());
    end;

    procedure LogPerformantEvent(EventId: Text; Message: Text; CustomDimensions: Dictionary of [Text, Text])
    var
        TelemetryData: Text;
    begin
        if IsBufferingEnabled then begin
            TelemetryData := SerializeTelemetryEvent(EventId, Message, CustomDimensions);
            AddToBuffer(TelemetryData);
        end else begin
            // Direct logging for debug scenarios
            Session.LogMessage(EventId, Message, Verbosity::Normal,
                DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
        end;
    end;

    local procedure AddToBuffer(TelemetryData: Text)
    begin
        TelemetryBuffer.Add(TelemetryData);
        
        if TelemetryBuffer.Count >= BatchSize then
            FlushTelemetryBuffer();
    end;

    procedure FlushTelemetryBuffer()
    var
        BatchData: Text;
        i: Integer;
    begin
        if TelemetryBuffer.Count = 0 then
            exit;
            
        for i := 1 to TelemetryBuffer.Count do begin
            BatchData := TelemetryBuffer.Get(i);
            ProcessBatchedTelemetryData(BatchData);
        end;
        
        TelemetryBuffer.Clear();
        LastFlushTime := CurrentDateTime;
    end;

    local procedure ProcessBatchedTelemetryData(TelemetryData: Text)
    var
        EventData: Dictionary of [Text, Text];
        CustomDimensions: Dictionary of [Text, Text];
    begin
        if DeserializeTelemetryEvent(TelemetryData, EventData) then begin
            CustomDimensions := GetCustomDimensionsFromEventData(EventData);
            Session.LogMessage(EventData.Get('EventId'), EventData.Get('Message'), Verbosity::Normal,
                DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
        end;
    end;

    local procedure IsDebugMode(): Boolean
    begin
        exit(false); // Implement debug mode detection
    end;

    local procedure SerializeTelemetryEvent(EventId: Text; Message: Text; CustomDimensions: Dictionary of [Text, Text]): Text
    begin
        // Simplified serialization - in production use proper JSON serialization
        exit(EventId + '|' + Message + '|' + SerializeCustomDimensions(CustomDimensions));
    end;

    local procedure DeserializeTelemetryEvent(TelemetryData: Text; var EventData: Dictionary of [Text, Text]): Boolean
    var
        Parts: List of [Text];
        DataPart: Text;
    begin
        Parts := TelemetryData.Split('|');
        if Parts.Count >= 3 then begin
            EventData.Add('EventId', Parts.Get(1));
            EventData.Add('Message', Parts.Get(2));
            exit(true);
        end;
        exit(false);
    end;

    local procedure GetCustomDimensionsFromEventData(EventData: Dictionary of [Text, Text]): Dictionary of [Text, Text]
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        // Extract and deserialize custom dimensions
        CustomDimensions.Add('Batched', 'true');
        exit(CustomDimensions);
    end;

    local procedure SerializeCustomDimensions(CustomDimensions: Dictionary of [Text, Text]): Text
    begin
        // Implement proper serialization logic
        exit('');
    end;
}
```

## Memory-Optimized Telemetry
```al
codeunit 50131 "Memory Optimized Telemetry"
{
    // Example 2: Memory-efficient telemetry for resource-constrained scenarios
    procedure LogMemoryOptimizedEvent(EventId: Text; MessageTemplate: Text; Parameters: List of [Text])
    var
        OptimizedMessage: Text;
        MinimalDimensions: Dictionary of [Text, Text];
    begin
        // Use string templates to reduce memory allocation
        OptimizedMessage := FormatMessageTemplate(MessageTemplate, Parameters);
        
        // Use only essential dimensions
        MinimalDimensions := GetEssentialDimensions();
        
        Session.LogMessage(EventId, OptimizedMessage, Verbosity::Normal,
            DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, MinimalDimensions);
    end;

    local procedure GetEssentialDimensions(): Dictionary of [Text, Text]
    var
        EssentialDimensions: Dictionary of [Text, Text];
    begin
        // Include only the most critical dimensions to minimize memory usage
        EssentialDimensions.Add('U', CopyStr(UserId(), 1, 20)); // Shortened key names
        EssentialDimensions.Add('C', CopyStr(CompanyName(), 1, 30));
        EssentialDimensions.Add('T', Format(CurrentDateTime, 0, '<Hours24><Minutes,2><Seconds,2>'));
        
        exit(EssentialDimensions);
    end;

    local procedure FormatMessageTemplate(Template: Text; Parameters: List of [Text]): Text
    var
        FormattedMessage: Text;
        Parameter: Text;
        ParameterIndex: Integer;
        PlaceholderText: Text;
    begin
        FormattedMessage := Template;
        ParameterIndex := 1;
        
        foreach Parameter in Parameters do begin
            PlaceholderText := '{' + Format(ParameterIndex) + '}';
            FormattedMessage := FormattedMessage.Replace(PlaceholderText, Parameter);
            ParameterIndex += 1;
        end;
        
        exit(FormattedMessage);
    end;

    // Example 3: Sampling for high-volume events
    procedure LogSampledEvent(EventId: Text; Message: Text; SampleRate: Decimal; CustomDimensions: Dictionary of [Text, Text])
    begin
        if ShouldSampleEvent(SampleRate) then
            Session.LogMessage(EventId, Message, Verbosity::Normal,
                DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
    end;

    local procedure ShouldSampleEvent(SampleRate: Decimal): Boolean
    var
        RandomValue: Decimal;
    begin
        if SampleRate >= 1.0 then
            exit(true);
            
        if SampleRate <= 0.0 then
            exit(false);
            
        RandomValue := Random(10000) / 10000.0;
        exit(RandomValue <= SampleRate);
    end;
}
```

## Conditional Performance Telemetry
```al
codeunit 50132 "Conditional Performance Telemetry"
{
    // Example 4: Performance-critical path telemetry
    procedure LogPerformanceCriticalEvent(EventCategory: Text; Message: Text; DetailedData: Dictionary of [Text, Text])
    var
        ShouldLog: Boolean;
        CustomDimensions: Dictionary of [Text, Text];
    begin
        ShouldLog := ShouldLogEvent(EventCategory);
        
        if ShouldLog then begin
            if IsHighPerformanceMode() then
                CustomDimensions := GetMinimalDimensions()
            else
                CustomDimensions := GetOptimizedDimensions(EventCategory);
                
            // Merge only essential detailed data in high-performance mode
            if not IsHighPerformanceMode() then
                MergeDetailedData(CustomDimensions, DetailedData)
            else
                MergeEssentialData(CustomDimensions, DetailedData);
                
            Session.LogMessage(GetEventId(EventCategory), Message, GetOptimalVerbosity(EventCategory),
                DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, CustomDimensions);
        end;
    end;

    local procedure IsHighPerformanceMode(): Boolean
    begin
        // Check system load indicators
        exit(GetCurrentSystemLoad() > 80);
    end;

    local procedure GetCurrentSystemLoad(): Integer
    begin
        // Simplified system load calculation
        // In real implementation, consider CPU, memory, and I/O metrics
        exit(Random(100));
    end;

    local procedure ShouldLogEvent(EventCategory: Text): Boolean
    begin
        case EventCategory of
            'Critical', 'Error':
                exit(true); // Always log critical events
            'Performance':
                exit(not IsHighPerformanceMode()); // Skip in high load
            'Debug':
                exit(false); // Skip debug in production
            else
                exit(true);
        end;
    end;

    local procedure GetMinimalDimensions(): Dictionary of [Text, Text]
    var
        MinimalDimensions: Dictionary of [Text, Text];
    begin
        MinimalDimensions.Add('UserID', UserId());
        MinimalDimensions.Add('Company', CompanyName());
        exit(MinimalDimensions);
    end;

    local procedure GetOptimizedDimensions(EventCategory: Text): Dictionary of [Text, Text]
    var
        OptimizedDimensions: Dictionary of [Text, Text];
    begin
        OptimizedDimensions.Add('UserID', UserId());
        OptimizedDimensions.Add('Company', CompanyName());
        OptimizedDimensions.Add('Category', EventCategory);
        OptimizedDimensions.Add('Timestamp', Format(CurrentDateTime, 0, 9));
        exit(OptimizedDimensions);
    end;

    local procedure MergeEssentialData(var CustomDimensions: Dictionary of [Text, Text]; DetailedData: Dictionary of [Text, Text])
    var
        Key: Text;
        EssentialKeys: List of [Text];
    begin
        // Define essential keys for high-performance scenarios
        EssentialKeys.Add('ErrorCode');
        EssentialKeys.Add('Duration');
        EssentialKeys.Add('RecordCount');
        EssentialKeys.Add('Status');
        
        foreach Key in EssentialKeys do
            if DetailedData.ContainsKey(Key) then
                CustomDimensions.Set(Key, DetailedData.Get(Key));
    end;

    local procedure MergeDetailedData(var CustomDimensions: Dictionary of [Text, Text]; DetailedData: Dictionary of [Text, Text])
    var
        Key: Text;
    begin
        foreach Key in DetailedData.Keys do
            CustomDimensions.Set(Key, DetailedData.Get(Key));
    end;

    local procedure GetOptimalVerbosity(EventCategory: Text): Verbosity
    begin
        case EventCategory of
            'Critical': exit(Verbosity::Critical);
            'Error': exit(Verbosity::Error);
            'Warning': exit(Verbosity::Warning);
            'Performance': 
                if IsHighPerformanceMode() then 
                    exit(Verbosity::Critical)
                else 
                    exit(Verbosity::Normal);
            else exit(Verbosity::Normal);
        end;
    end;

    local procedure GetEventId(Category: Text): Text
    begin
        exit(Category + Format(Random(999), 3, '<Integer,3><Filler Character,0>'));
    end;
}
```