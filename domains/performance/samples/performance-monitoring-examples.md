# Performance Monitoring Examples

## Basic Performance Monitoring Setup

### Performance Metrics Collection
```al
table 50400 "Performance Metrics Log"
{
    Caption = 'Performance Metrics Log';
    DataClassification = SystemMetadata;
    
    fields
    {
        field(1; "Entry No."; Integer) { AutoIncrement = true; }
        field(2; "Operation Type"; Text[50]) { }
        field(3; "Operation Details"; Text[250]) { }
        field(4; "Start Time"; DateTime) { }
        field(5; "End Time"; DateTime) { }
        field(6; "Duration (ms)"; Integer) { }
        field(7; "Records Processed"; Integer) { }
        field(8; "Memory Usage (KB)"; Integer) { }
        field(9; "CPU Usage %"; Decimal) { }
        field(10; "User ID"; Text[50]) { }
        field(11; "Session ID"; Integer) { }
        field(12; "Company Name"; Text[30]) { }
    }
    
    keys
    {
        key(PK; "Entry No.") { Clustered = true; }
        key(Performance; "Operation Type", "Start Time") { }
        key(Duration; "Duration (ms)") { }
    }
}

codeunit 50400 "Performance Monitor"
{
    var
        GlobalPerformanceMetrics: Record "Performance Metrics Log";
        
    procedure StartOperation(OperationType: Text[50]; OperationDetails: Text[250]): Integer
    var
        PerformanceMetrics: Record "Performance Metrics Log";
    begin
        // ✅ MONITORING: Start performance tracking
        PerformanceMetrics.Init();
        PerformanceMetrics."Operation Type" := OperationType;
        PerformanceMetrics."Operation Details" := OperationDetails;
        PerformanceMetrics."Start Time" := CurrentDateTime();
        PerformanceMetrics."User ID" := UserId();
        PerformanceMetrics."Session ID" := SessionId();
        PerformanceMetrics."Company Name" := CompanyName();
        PerformanceMetrics.Insert();
        
        exit(PerformanceMetrics."Entry No.");
    end;
    
    procedure EndOperation(EntryNo: Integer; RecordsProcessed: Integer)
    var
        PerformanceMetrics: Record "Performance Metrics Log";
        Duration: Duration;
    begin
        // ✅ MONITORING: Complete performance tracking
        if PerformanceMetrics.Get(EntryNo) then begin
            PerformanceMetrics."End Time" := CurrentDateTime();
            Duration := PerformanceMetrics."End Time" - PerformanceMetrics."Start Time";
            PerformanceMetrics."Duration (ms)" := Duration;
            PerformanceMetrics."Records Processed" := RecordsProcessed;
            PerformanceMetrics."Memory Usage (KB)" := GetCurrentMemoryUsage();
            PerformanceMetrics."CPU Usage %" := GetCurrentCPUUsage();
            PerformanceMetrics.Modify();
            
            // Alert on performance thresholds
            CheckPerformanceThresholds(PerformanceMetrics);
        end;
    end;
    
    local procedure GetCurrentMemoryUsage(): Integer
    begin
        // ✅ SYSTEM METRICS: Get current memory usage
        // Implementation would call system APIs
        // For example purposes, return simulated value
        exit(Random(100000)); // KB
    end;
    
    local procedure GetCurrentCPUUsage(): Decimal
    begin
        // ✅ SYSTEM METRICS: Get current CPU usage
        // Implementation would call system APIs
        exit(Random(100)); // Percentage
    end;
    
    local procedure CheckPerformanceThresholds(PerformanceMetrics: Record "Performance Metrics Log")
    var
        ThresholdExceeded: Boolean;
        AlertMessage: Text;
    begin
        // ✅ ALERTING: Check performance thresholds
        case PerformanceMetrics."Operation Type" of
            'REPORT_GENERATION':
                ThresholdExceeded := PerformanceMetrics."Duration (ms)" > 30000; // 30 seconds
            'BATCH_PROCESSING':
                ThresholdExceeded := PerformanceMetrics."Duration (ms)" > 300000; // 5 minutes
            'PAGE_LOAD':
                ThresholdExceeded := PerformanceMetrics."Duration (ms)" > 2000; // 2 seconds
            'FLOWFIELD_CALC':
                ThresholdExceeded := PerformanceMetrics."Duration (ms)" > 1000; // 1 second
            else
                ThresholdExceeded := PerformanceMetrics."Duration (ms)" > 10000; // 10 seconds default
        end;
        
        if ThresholdExceeded then begin
            AlertMessage := StrSubstNo('Performance threshold exceeded:\Operation: %1\Duration: %2ms\Details: %3',
                                     PerformanceMetrics."Operation Type",
                                     PerformanceMetrics."Duration (ms)",
                                     PerformanceMetrics."Operation Details");
            
            LogPerformanceAlert(AlertMessage, PerformanceMetrics);
            
            // Send notification to administrators
            SendPerformanceAlert(AlertMessage);
        end;
    end;
    
    local procedure LogPerformanceAlert(AlertMessage: Text; PerformanceMetrics: Record "Performance Metrics Log")
    var
        ActivityLog: Record "Activity Log";
    begin
        ActivityLog.LogActivity(
            PerformanceMetrics.RecordId(),
            ActivityLog.Status::Failed,
            'PERFORMANCE',
            AlertMessage,
            StrSubstNo('Duration: %1ms, Records: %2', PerformanceMetrics."Duration (ms)", PerformanceMetrics."Records Processed")
        );
    end;
    
    local procedure SendPerformanceAlert(AlertMessage: Text)
    begin
        // ✅ NOTIFICATION: Send alert to system administrators
        // Implementation would send email, Teams notification, etc.
        Message('PERFORMANCE ALERT: %1', AlertMessage);
    end;
}
```

## Advanced Performance Analysis

### Performance Trend Analysis
```al
codeunit 50401 "Performance Analyzer"
{
    procedure AnalyzePerformanceTrends(OperationType: Text[50]; DaysBack: Integer): JsonObject
    var
        PerformanceMetrics: Record "Performance Metrics Log";
        TrendAnalysis: JsonObject;
        DailyStats: JsonArray;
        CurrentDate: Date;
        DayStats: JsonObject;
        TotalDuration: Integer;
        Count: Integer;
        AvgDuration: Decimal;
    begin
        // ✅ TREND ANALYSIS: Analyze performance over time
        PerformanceMetrics.SetRange("Operation Type", OperationType);
        PerformanceMetrics.SetFilter("Start Time", '>=%1', CurrentDateTime() - (DaysBack * 86400000));
        PerformanceMetrics.SetCurrentKey("Operation Type", "Start Time");
        
        if PerformanceMetrics.FindSet() then begin
            repeat
                if CurrentDate <> DT2Date(PerformanceMetrics."Start Time") then begin
                    // Store previous day's stats
                    if CurrentDate <> 0D then
                        AddDayStats(CurrentDate, Count, TotalDuration, DailyStats);
                    
                    // Start new day
                    CurrentDate := DT2Date(PerformanceMetrics."Start Time");
                    TotalDuration := 0;
                    Count := 0;
                end;
                
                TotalDuration += PerformanceMetrics."Duration (ms)";
                Count += 1;
                
            until PerformanceMetrics.Next() = 0;
            
            // Add final day
            if CurrentDate <> 0D then
                AddDayStats(CurrentDate, Count, TotalDuration, DailyStats);
        end;
        
        // Calculate overall average
        PerformanceMetrics.Reset();
        PerformanceMetrics.SetRange("Operation Type", OperationType);
        PerformanceMetrics.SetFilter("Start Time", '>=%1', CurrentDateTime() - (DaysBack * 86400000));
        
        if PerformanceMetrics.FindSet() then begin
            TotalDuration := 0;
            Count := 0;
            repeat
                TotalDuration += PerformanceMetrics."Duration (ms)";
                Count += 1;
            until PerformanceMetrics.Next() = 0;
            
            if Count > 0 then
                AvgDuration := TotalDuration / Count;
        end;
        
        TrendAnalysis.Add('operation_type', OperationType);
        TrendAnalysis.Add('analysis_period_days', DaysBack);
        TrendAnalysis.Add('total_operations', Count);
        TrendAnalysis.Add('average_duration_ms', AvgDuration);
        TrendAnalysis.Add('daily_statistics', DailyStats);
        TrendAnalysis.Add('analysis_timestamp', CurrentDateTime());
        
        exit(TrendAnalysis);
    end;
    
    local procedure AddDayStats(CurrentDate: Date; Count: Integer; TotalDuration: Integer; var DailyStats: JsonArray)
    var
        DayStats: JsonObject;
    begin
        DayStats.Add('date', CurrentDate);
        DayStats.Add('operation_count', Count);
        DayStats.Add('total_duration_ms', TotalDuration);
        if Count > 0 then
            DayStats.Add('average_duration_ms', TotalDuration / Count)
        else
            DayStats.Add('average_duration_ms', 0);
        
        DailyStats.Add(DayStats);
    end;
    
    procedure GetTopSlowOperations(TopCount: Integer): JsonArray
    var
        PerformanceMetrics: Record "Performance Metrics Log";
        SlowOperations: JsonArray;
        OperationJson: JsonObject;
        Counter: Integer;
    begin
        // ✅ SLOW QUERY IDENTIFICATION: Find slowest operations
        PerformanceMetrics.SetCurrentKey("Duration (ms)");
        PerformanceMetrics.Ascending(false);
        PerformanceMetrics.SetFilter("Start Time", '>=%1', CurrentDateTime() - 86400000); // Last 24 hours
        
        if PerformanceMetrics.FindSet() then begin
            repeat
                Clear(OperationJson);
                OperationJson.Add('entry_no', PerformanceMetrics."Entry No.");
                OperationJson.Add('operation_type', PerformanceMetrics."Operation Type");
                OperationJson.Add('operation_details', PerformanceMetrics."Operation Details");
                OperationJson.Add('duration_ms', PerformanceMetrics."Duration (ms)");
                OperationJson.Add('records_processed', PerformanceMetrics."Records Processed");
                OperationJson.Add('start_time', PerformanceMetrics."Start Time");
                OperationJson.Add('user_id', PerformanceMetrics."User ID");
                
                SlowOperations.Add(OperationJson);
                Counter += 1;
                
            until (PerformanceMetrics.Next() = 0) or (Counter >= TopCount);
        end;
        
        exit(SlowOperations);
    end;
    
    procedure GeneratePerformanceReport(): Text
    var
        PerformanceMetrics: Record "Performance Metrics Log";
        ReportBuilder: TextBuilder;
        OperationType: Text[50];
        Count: Integer;
        TotalDuration: Integer;
        AvgDuration: Decimal;
        MaxDuration: Integer;
    begin
        // ✅ COMPREHENSIVE REPORT: Generate performance summary
        ReportBuilder.AppendLine('=== PERFORMANCE ANALYSIS REPORT ===');
        ReportBuilder.AppendLine(StrSubstNo('Generated: %1', CurrentDateTime()));
        ReportBuilder.AppendLine(StrSubstNo('Analysis Period: Last 24 hours'));
        ReportBuilder.AppendLine('');
        
        // Group by operation type
        PerformanceMetrics.SetFilter("Start Time", '>=%1', CurrentDateTime() - 86400000);
        PerformanceMetrics.SetCurrentKey("Operation Type", "Start Time");
        
        if PerformanceMetrics.FindSet() then begin
            repeat
                if OperationType <> PerformanceMetrics."Operation Type" then begin
                    // Output previous operation stats
                    if OperationType <> '' then
                        AddOperationStats(OperationType, Count, TotalDuration, MaxDuration, ReportBuilder);
                    
                    // Start new operation type
                    OperationType := PerformanceMetrics."Operation Type";
                    Count := 0;
                    TotalDuration := 0;
                    MaxDuration := 0;
                end;
                
                Count += 1;
                TotalDuration += PerformanceMetrics."Duration (ms)";
                if PerformanceMetrics."Duration (ms)" > MaxDuration then
                    MaxDuration := PerformanceMetrics."Duration (ms)";
                
            until PerformanceMetrics.Next() = 0;
            
            // Add final operation
            if OperationType <> '' then
                AddOperationStats(OperationType, Count, TotalDuration, MaxDuration, ReportBuilder);
        end;
        
        ReportBuilder.AppendLine('');
        ReportBuilder.AppendLine('=== END REPORT ===');
        
        exit(ReportBuilder.ToText());
    end;
    
    local procedure AddOperationStats(OperationType: Text[50]; Count: Integer; TotalDuration: Integer; MaxDuration: Integer; var ReportBuilder: TextBuilder)
    var
        AvgDuration: Decimal;
    begin
        if Count > 0 then
            AvgDuration := TotalDuration / Count;
        
        ReportBuilder.AppendLine(StrSubstNo('Operation: %1', OperationType));
        ReportBuilder.AppendLine(StrSubstNo('  Count: %1', Count));
        ReportBuilder.AppendLine(StrSubstNo('  Total Duration: %1ms', TotalDuration));
        ReportBuilder.AppendLine(StrSubstNo('  Average Duration: %1ms', Round(AvgDuration, 1)));
        ReportBuilder.AppendLine(StrSubstNo('  Max Duration: %1ms', MaxDuration));
        ReportBuilder.AppendLine('');
    end;
}
```

## Real-time Performance Monitoring

### Performance Dashboard Data Provider
```al
codeunit 50402 "Performance Dashboard"
{
    procedure GetCurrentSystemMetrics(): JsonObject
    var
        SystemMetrics: JsonObject;
        ActiveSessions: Integer;
        CurrentCPU: Decimal;
        CurrentMemory: Integer;
        DatabaseConnections: Integer;
    begin
        // ✅ REAL-TIME METRICS: Collect current system state
        ActiveSessions := GetActiveSessionCount();
        CurrentCPU := GetCurrentCPUUsage();
        CurrentMemory := GetCurrentMemoryUsage();
        DatabaseConnections := GetDatabaseConnectionCount();
        
        SystemMetrics.Add('timestamp', CurrentDateTime());
        SystemMetrics.Add('active_sessions', ActiveSessions);
        SystemMetrics.Add('cpu_usage_percent', CurrentCPU);
        SystemMetrics.Add('memory_usage_mb', CurrentMemory / 1024);
        SystemMetrics.Add('database_connections', DatabaseConnections);
        SystemMetrics.Add('system_status', DetermineSystemStatus(CurrentCPU, CurrentMemory));
        
        exit(SystemMetrics);
    end;
    
    local procedure GetActiveSessionCount(): Integer
    var
        ActiveSession: Record "Active Session";
    begin
        // ✅ SESSION COUNT: Get current active sessions
        exit(ActiveSession.Count());
    end;
    
    local procedure GetDatabaseConnectionCount(): Integer
    begin
        // ✅ DATABASE METRICS: Get current DB connections
        // Implementation would query SQL Server DMVs
        exit(Random(50) + 10); // Simulated for example
    end;
    
    local procedure DetermineSystemStatus(CPUUsage: Decimal; MemoryUsage: Integer): Text
    begin
        // ✅ STATUS DETERMINATION: Calculate overall system health
        case true of
            (CPUUsage > 90) or (MemoryUsage > 1000000):
                exit('CRITICAL');
            (CPUUsage > 75) or (MemoryUsage > 800000):
                exit('WARNING');
            (CPUUsage > 50) or (MemoryUsage > 600000):
                exit('MODERATE');
            else
                exit('HEALTHY');
        end;
    end;
    
    procedure GetRecentPerformanceAlerts(): JsonArray
    var
        PerformanceMetrics: Record "Performance Metrics Log";
        AlertsArray: JsonArray;
        AlertJson: JsonObject;
    begin
        // ✅ RECENT ALERTS: Get performance issues from last hour
        PerformanceMetrics.SetFilter("Start Time", '>=%1', CurrentDateTime() - 3600000); // Last hour
        PerformanceMetrics.SetFilter("Duration (ms)", '>%1', 5000); // Over 5 seconds
        PerformanceMetrics.SetCurrentKey("Duration (ms)");
        PerformanceMetrics.Ascending(false);
        
        if PerformanceMetrics.FindSet() then begin
            repeat
                Clear(AlertJson);
                AlertJson.Add('timestamp', PerformanceMetrics."Start Time");
                AlertJson.Add('operation', PerformanceMetrics."Operation Type");
                AlertJson.Add('duration_ms', PerformanceMetrics."Duration (ms)");
                AlertJson.Add('details', PerformanceMetrics."Operation Details");
                AlertJson.Add('user', PerformanceMetrics."User ID");
                AlertJson.Add('severity', GetAlertSeverity(PerformanceMetrics."Duration (ms)"));
                
                AlertsArray.Add(AlertJson);
                
            until (PerformanceMetrics.Next() = 0) or (AlertsArray.Count() >= 10);
        end;
        
        exit(AlertsArray);
    end;
    
    local procedure GetAlertSeverity(Duration: Integer): Text
    begin
        case true of
            Duration > 60000:
                exit('CRITICAL');
            Duration > 30000:
                exit('HIGH');
            Duration > 10000:
                exit('MEDIUM');
            else
                exit('LOW');
        end;
    end;
}
```

## Usage Examples in Business Logic

### Monitored Business Process
```al
codeunit 50403 "Monitored Sales Processor"
{
    var
        PerformanceMonitor: Codeunit "Performance Monitor";
        
    procedure ProcessSalesOrderWithMonitoring(var SalesHeader: Record "Sales Header")
    var
        MonitoringID: Integer;
        ProcessedLines: Integer;
    begin
        // ✅ START MONITORING: Begin performance tracking
        MonitoringID := PerformanceMonitor.StartOperation(
            'SALES_ORDER_PROCESSING',
            StrSubstNo('Order: %1, Customer: %2', SalesHeader."No.", SalesHeader."Sell-to Customer No.")
        );
        
        try
            // Execute business logic
            ProcessedLines := ProcessSalesOrderLines(SalesHeader);
            CalculateOrderTotals(SalesHeader);
            ValidateOrderConstraints(SalesHeader);
            
        finally
            // ✅ END MONITORING: Complete performance tracking
            PerformanceMonitor.EndOperation(MonitoringID, ProcessedLines);
        end;
    end;
    
    local procedure ProcessSalesOrderLines(var SalesHeader: Record "Sales Header"): Integer
    var
        SalesLine: Record "Sales Line";
        ProcessedCount: Integer;
    begin
        SalesLine.SetRange("Document Type", SalesHeader."Document Type");
        SalesLine.SetRange("Document No.", SalesHeader."No.");
        
        if SalesLine.FindSet() then begin
            repeat
                ProcessSingleSalesLine(SalesLine);
                ProcessedCount += 1;
            until SalesLine.Next() = 0;
        end;
        
        exit(ProcessedCount);
    end;
    
    local procedure ProcessSingleSalesLine(var SalesLine: Record "Sales Line")
    begin
        // Business logic for processing individual sales line
        SalesLine.Validate("Unit Price");
        SalesLine.Modify();
    end;
    
    local procedure CalculateOrderTotals(var SalesHeader: Record "Sales Header")
    begin
        // Business logic for calculating order totals
        SalesHeader.CalcFields(Amount, "Amount Including VAT");
    end;
    
    local procedure ValidateOrderConstraints(var SalesHeader: Record "Sales Header")
    begin
        // Business logic for order validation
        SalesHeader.TestField("Sell-to Customer No.");
        SalesHeader.TestField("Order Date");
    end;
}
```