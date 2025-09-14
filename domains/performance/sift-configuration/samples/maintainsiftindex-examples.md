# MaintainSIFTIndex Property Examples

## Real-time SIFT Configuration (MaintainSIFTIndex=Yes)

### Dashboard-Optimized Table
```al
table 50100 "Sales Dashboard Data"
{
    Caption = 'Sales Dashboard Data';
    DataClassification = CustomerContent;

    fields
    {
        field(1; "Entry No."; Integer) { AutoIncrement = true; }
        field(10; "Customer No."; Code[20]) { TableRelation = Customer; }
        field(20; "Sales Amount"; Decimal) { DecimalPlaces = 2 : 2; }
        field(30; "Quantity"; Decimal) { DecimalPlaces = 0 : 5; }
        field(40; "Posting Date"; Date) { }
    }

    keys
    {
        key(PK; "Entry No.") { Clustered = true; }
        
        // Real-time SIFT for dashboard performance
        key(CustomerSIFT; "Customer No.")
        {
            SumIndexFields = "Sales Amount", Quantity;
            MaintainSIFTIndex = true; // Updated in real-time for instant dashboard access
        }
    }
}

// Dashboard page leveraging real-time SIFT
page 50100 "Sales Dashboard"
{
    PageType = RoleCenter;
    SourceTable = Customer;

    layout
    {
        area(content)
        {
            cuegroup("Sales Performance")
            {
                field("Total Sales"; GetCustomerSales())
                {
                    Caption = 'Total Sales';
                    // Leverages real-time SIFT - no calculation delay
                }
            }
        }
    }

    local procedure GetCustomerSales(): Decimal
    var
        SalesDashboard: Record "Sales Dashboard Data";
    begin
        SalesDashboard.SetRange("Customer No.", Rec."No.");
        SalesDashboard.CalcSums("Sales Amount"); // Uses real-time SIFT
        exit(SalesDashboard."Sales Amount");
    end;
}
```

## Batch Processing Optimized (MaintainSIFTIndex=No)

### Import-Heavy Data Table
```al
table 50101 "Import Processing Data"
{
    Caption = 'Import Processing Data';
    DataClassification = CustomerContent;

    fields
    {
        field(1; ID; Integer) { AutoIncrement = true; }
        field(10; "Batch ID"; Code[20]) { }
        field(20; Amount; Decimal) { DecimalPlaces = 2 : 2; }
        field(30; "Process Date"; Date) { }
    }

    keys
    {
        key(PK; ID) { Clustered = true; }
        
        // On-demand SIFT for batch import optimization
        key(BatchSIFT; "Batch ID")
        {
            SumIndexFields = Amount;
            MaintainSIFTIndex = false; // No real-time overhead during imports
        }
    }
}

// Fast batch import with deferred aggregation
codeunit 50100 "Batch Import Manager"
{
    procedure ProcessLargeBatch(BatchID: Code[20]; RecordCount: Integer)
    var
        ImportData: Record "Import Processing Data";
        i: Integer;
        StartTime: DateTime;
        ImportDuration: Duration;
        TotalAmount: Decimal;
    begin
        StartTime := CurrentDateTime;
        
        // Fast import without SIFT maintenance overhead
        for i := 1 to RecordCount do begin
            ImportData.Init();
            ImportData."Batch ID" := BatchID;
            ImportData.Amount := Random(1000);
            ImportData."Process Date" := Today;
            ImportData.Insert(); // No SIFT updates during import
        end;
        
        ImportDuration := CurrentDateTime - StartTime;
        
        // Calculate totals using on-demand SIFT after import
        ImportData.Reset();
        ImportData.SetRange("Batch ID", BatchID);
        ImportData.CalcSums(Amount); // SIFT built on first calculation
        TotalAmount := ImportData.Amount;
        
        Message('Imported %1 records in %2ms\nTotal Amount: %3',
                RecordCount, ImportDuration, TotalAmount);
    end;
}
```

## Dynamic SIFT Management

### Context-Aware SIFT Configuration
```al
table 50102 "Flexible Analytics Data"
{
    Caption = 'Flexible Analytics Data';
    DataClassification = CustomerContent;

    fields
    {
        field(1; ID; Integer) { AutoIncrement = true; }
        field(10; "Department Code"; Code[10]) { }
        field(20; "Transaction Amount"; Decimal) { }
        field(30; "Transaction Date"; Date) { }
    }

    keys
    {
        key(PK; ID) { Clustered = true; }
        
        // SIFT configuration varies by usage mode
        key(DepartmentAnalytics; "Department Code")
        {
            SumIndexFields = "Transaction Amount";
            // MaintainSIFTIndex set dynamically based on system mode
        }
    }
}

// Dynamic SIFT management based on system state
codeunit 50101 "Dynamic SIFT Manager"
{
    procedure SetSIFTModeForReporting()
    begin
        // Enable real-time SIFT during reporting periods
        SetTableSIFTMode(Database::"Flexible Analytics Data", true);
        Message('SIFT enabled for real-time reporting');
    end;
    
    procedure SetSIFTModeForDataEntry()
    begin
        // Disable real-time SIFT during data entry periods
        SetTableSIFTMode(Database::"Flexible Analytics Data", false);
        Message('SIFT disabled for fast data entry');
    end;
    
    local procedure SetTableSIFTMode(TableID: Integer; EnableRealTime: Boolean)
    var
        TableMetadata: Record "Table Metadata";
        KeyRef: KeyRef;
        RecRef: RecordRef;
    begin
        RecRef.Open(TableID);
        KeyRef := RecRef.KeyIndex(2); // Department Analytics key
        
        if EnableRealTime then
            KeyRef.SetProperty(Property::MaintainSIFTIndex, true)
        else
            KeyRef.SetProperty(Property::MaintainSIFTIndex, false);
            
        RecRef.Close();
    end;
}
```

## Performance Comparison Implementation

### SIFT Impact Measurement
```al
table 50103 "SIFT Performance Log"
{
    Caption = 'SIFT Performance Log';
    DataClassification = CustomerContent;

    fields
    {
        field(1; "Log Entry No."; Integer) { AutoIncrement = true; }
        field(10; "Test Type"; Option) { OptionMembers = "With SIFT","Without SIFT"; }
        field(20; "Execution Time (ms)"; Integer) { }
        field(30; "Record Count"; Integer) { }
        field(40; "Memory Usage (MB)"; Decimal) { }
        field(50; "Test Date"; DateTime) { }
    }

    keys
    {
        key(PK; "Log Entry No.") { }
        key(TestAnalysis; "Test Type", "Test Date") { }
    }
}

// Performance testing framework
codeunit 50102 "SIFT Performance Tester"
{
    procedure CompareSIFTPerformance(TestRecords: Integer)
    var
        TestDataWithSIFT: Record "Sales Dashboard Data";
        TestDataWithoutSIFT: Record "Import Processing Data";
        PerformanceLog: Record "SIFT Performance Log";
        StartTime: DateTime;
        EndTime: DateTime;
        TestAmount: Decimal;
    begin
        // Test 1: With real-time SIFT
        StartTime := CurrentDateTime;
        TestDataWithSIFT.Reset();
        TestDataWithSIFT.CalcSums("Sales Amount");
        TestAmount := TestDataWithSIFT."Sales Amount";
        EndTime := CurrentDateTime;
        
        LogPerformanceResult(PerformanceLog."Test Type"::"With SIFT", 
                           EndTime - StartTime, TestRecords);
        
        // Test 2: Without real-time SIFT (forces calculation)
        StartTime := CurrentDateTime;
        TestDataWithoutSIFT.Reset();
        TestDataWithoutSIFT.CalcSums(Amount); // Triggers on-demand calculation
        TestAmount := TestDataWithoutSIFT.Amount;
        EndTime := CurrentDateTime;
        
        LogPerformanceResult(PerformanceLog."Test Type"::"Without SIFT", 
                           EndTime - StartTime, TestRecords);
    end;
    
    local procedure LogPerformanceResult(TestType: Option; Duration: Duration; RecordCount: Integer)
    var
        PerformanceLog: Record "SIFT Performance Log";
    begin
        PerformanceLog.Init();
        PerformanceLog."Test Type" := TestType;
        PerformanceLog."Execution Time (ms)" := Duration;
        PerformanceLog."Record Count" := RecordCount;
        PerformanceLog."Test Date" := CurrentDateTime;
        PerformanceLog.Insert();
    end;
}
```

## Best Practice Decision Examples

### Workload-Based Configuration
```al
// Configuration patterns for different scenarios
codeunit 50103 "SIFT Configuration Patterns"
{
    // Pattern 1: High-frequency dashboard
    procedure ConfigureForDashboard()
    var
        ConfigNote: Text;
    begin
        ConfigNote := 'Dashboard Pattern: MaintainSIFTIndex = true\n' +
                     '- Read/Write Ratio: 50:1\n' +
                     '- Update Frequency: Every few minutes\n' +
                     '- Query Frequency: Constant (dashboard refresh)\n' +
                     '- Justification: Real-time updates essential for UX';
        Message(ConfigNote);
    end;
    
    // Pattern 2: Batch processing system
    procedure ConfigureForBatchProcessing()
    var
        ConfigNote: Text;
    begin
        ConfigNote := 'Batch Pattern: MaintainSIFTIndex = false\n' +
                     '- Read/Write Ratio: 1:100\n' +
                     '- Update Frequency: Continuous during batch\n' +
                     '- Query Frequency: After batch completion\n' +
                     '- Justification: Import speed more important than query speed';
        Message(ConfigNote);
    end;
    
    // Pattern 3: Mixed-mode system
    procedure ConfigureForMixedMode()
    var
        ConfigNote: Text;
    begin
        ConfigNote := 'Mixed Pattern: Time-based switching\n' +
                     '- Business hours: MaintainSIFTIndex = true (reporting)\n' +
                     '- Off hours: MaintainSIFTIndex = false (batch jobs)\n' +
                     '- Scheduled switching based on business cycles\n' +
                     '- Justification: Optimize for primary usage per time period';
        Message(ConfigNote);
    end;
}

/*
MaintainSIFTIndex Decision Framework:

1. Analysis Questions:
   - What is the read-to-write ratio for this data?
   - How frequently are aggregations requested?
   - Are there specific time periods with different usage patterns?
   - What are the performance requirements for both reads and writes?

2. Configuration Guidelines:
   - Read-heavy (10:1 ratio or higher): MaintainSIFTIndex = true
   - Write-heavy (1:10 ratio or higher): MaintainSIFTIndex = false
   - Balanced (1:1 to 10:1): Test both and measure actual performance
   - Time-sensitive queries: MaintainSIFTIndex = true
   - Batch processing: MaintainSIFTIndex = false

3. Monitoring and Adjustment:
   - Measure actual performance in production-like environments
   - Monitor both query response times and data modification speeds
   - Review usage patterns quarterly
   - Adjust configuration as business processes evolve
*/
```