# SIFT Technology Fundamentals - AL Code Samples

## Basic SIFT Table Configuration

### Standard Implementation
```al
table 50100 "Sales Analytics Data"
{
    Caption = 'Sales Analytics Data';
    DataClassification = CustomerContent;

    fields
    {
        field(1; "Customer No."; Code[20])
        {
            Caption = 'Customer No.';
            TableRelation = Customer;
        }
        field(2; "Item No."; Code[20])
        {
            Caption = 'Item No.';
            TableRelation = Item;
        }
        field(3; "Posting Date"; Date)
        {
            Caption = 'Posting Date';
        }
        field(10; "Sales Amount"; Decimal)
        {
            Caption = 'Sales Amount';
            DecimalPlaces = 2 : 2;
        }
        field(11; "Quantity"; Decimal)
        {
            Caption = 'Quantity';
            DecimalPlaces = 0 : 5;
        }
        field(12; "Profit Amount"; Decimal)
        {
            Caption = 'Profit Amount';
            DecimalPlaces = 2 : 2;
        }
    }

    keys
    {
        key(PK; "Customer No.", "Item No.", "Posting Date")
        {
            Clustered = true;
        }
        
        // Basic SIFT key for customer totals
        key(CustomerSIFT; "Customer No.")
        {
            SumIndexFields = "Sales Amount", Quantity, "Profit Amount";
            MaintainSIFTIndex = true;
        }
        
        // Item-based aggregations
        key(ItemSIFT; "Item No.")
        {
            SumIndexFields = "Sales Amount", Quantity;
            MaintainSIFTIndex = true;
        }
    }
}
```

## SumIndexFields Property Usage Patterns

### Multiple Aggregation Strategies
```al
table 50101 "Order Performance Metrics"
{
    Caption = 'Order Performance Metrics';
    DataClassification = CustomerContent;

    fields
    {
        field(1; "Salesperson Code"; Code[20]) { }
        field(2; "Customer No."; Code[20]) { }
        field(3; "Order Date"; Date) { }
        field(10; "Order Amount"; Decimal) { }
        field(11; "Discount Amount"; Decimal) { }
        field(12; "Commission Amount"; Decimal) { }
        field(13; "Processing Time"; Integer) { }
    }

    keys
    {
        key(PK; "Salesperson Code", "Customer No.", "Order Date") { Clustered = true; }
        
        // Comprehensive salesperson performance SIFT
        key(SalespersonPerformance; "Salesperson Code")
        {
            SumIndexFields = "Order Amount", "Discount Amount", "Commission Amount";
            MaintainSIFTIndex = true;
        }
        
        // Customer relationship metrics SIFT
        key(CustomerMetrics; "Customer No.")
        {
            SumIndexFields = "Order Amount", "Discount Amount";
            MaintainSIFTIndex = true;
        }
        
        // Time-based analysis SIFT
        key(DateAnalysis; "Order Date")
        {
            SumIndexFields = "Order Amount", "Processing Time";
            MaintainSIFTIndex = false; // On-demand for reporting
        }
    }
}

// Usage in AL code
codeunit 50100 "SIFT Usage Examples"
{
    procedure GetSalespersonTotals(SalespersonCode: Code[20]; var TotalAmount: Decimal; var TotalCommission: Decimal)
    var
        OrderMetrics: Record "Order Performance Metrics";
    begin
        OrderMetrics.Reset();
        OrderMetrics.SetRange("Salesperson Code", SalespersonCode);
        
        // Leverages SIFT for instant calculation
        OrderMetrics.CalcSums("Order Amount", "Commission Amount");
        TotalAmount := OrderMetrics."Order Amount";
        TotalCommission := OrderMetrics."Commission Amount";
    end;
}
```

## FlowField Integration with SIFT

### Customer Card with SIFT-Optimized FlowFields
```al
tableextension 50100 "Customer SIFT Extension" extends Customer
{
    fields
    {
        // FlowField automatically leverages SIFT when available
        field(50100; "Total Sales Amount"; Decimal)
        {
            Caption = 'Total Sales Amount';
            FieldClass = FlowField;
            CalcFormula = sum("Sales Analytics Data"."Sales Amount" where("Customer No." = field("No.")));
            Editable = false;
        }
        
        field(50101; "Total Quantity Sold"; Decimal)
        {
            Caption = 'Total Quantity Sold';
            FieldClass = FlowField;
            CalcFormula = sum("Sales Analytics Data".Quantity where("Customer No." = field("No.")));
            Editable = false;
        }
        
        field(50102; "Average Order Value"; Decimal)
        {
            Caption = 'Average Order Value';
            FieldClass = FlowField;
            CalcFormula = average("Sales Analytics Data"."Sales Amount" where("Customer No." = field("No.")));
            Editable = false;
        }
    }
}

// Page implementation with automatic FlowField calculation
pageextension 50100 "Customer Card SIFT" extends "Customer Card"
{
    layout
    {
        addafter("Name")
        {
            group("Sales Analytics")
            {
                Caption = 'Sales Analytics (SIFT-Optimized)';
                
                field("Total Sales Amount"; Rec."Total Sales Amount")
                {
                    ApplicationArea = All;
                    // Automatically calculated using SIFT when page opens
                }
                field("Total Quantity Sold"; Rec."Total Quantity Sold")
                {
                    ApplicationArea = All;
                }
                field("Average Order Value"; Rec."Average Order Value")
                {
                    ApplicationArea = All;
                }
            }
        }
    }
    
    trigger OnAfterGetRecord()
    begin
        // Force calculation of FlowFields (leverages SIFT)
        Rec.CalcFields("Total Sales Amount", "Total Quantity Sold", "Average Order Value");
    end;
}
```

## Performance Comparison: SIFT vs Non-SIFT

### Performance Testing Framework
```al
codeunit 50101 "SIFT Performance Comparison"
{
    procedure ComparePerformance(CustomerNo: Code[20])
    var
        SalesData: Record "Sales Analytics Data";
        StartTime: DateTime;
        EndTime: DateTime;
        SIFTDuration: Duration;
        ManualDuration: Duration;
        TotalAmount: Decimal;
    begin
        // Test 1: SIFT-optimized calculation
        StartTime := CurrentDateTime;
        TotalAmount := CalculateUsingSIFT(CustomerNo);
        EndTime := CurrentDateTime;
        SIFTDuration := EndTime - StartTime;
        
        // Test 2: Manual calculation without SIFT
        StartTime := CurrentDateTime;
        TotalAmount := CalculateManually(CustomerNo);
        EndTime := CurrentDateTime;
        ManualDuration := EndTime - StartTime;
        
        // Performance comparison results
        Message('SIFT Calculation: %1ms\Manual Calculation: %2ms\Performance Improvement: %3x faster',
                SIFTDuration, ManualDuration, ManualDuration / SIFTDuration);
    end;
    
    local procedure CalculateUsingSIFT(CustomerNo: Code[20]): Decimal
    var
        SalesData: Record "Sales Analytics Data";
    begin
        // Leverages SIFT index for instant aggregation
        SalesData.Reset();
        SalesData.SetRange("Customer No.", CustomerNo);
        SalesData.CalcSums("Sales Amount");
        exit(SalesData."Sales Amount");
    end;
    
    local procedure CalculateManually(CustomerNo: Code[20]): Decimal
    var
        SalesData: Record "Sales Analytics Data";
        TotalAmount: Decimal;
    begin
        // Forces row-by-row calculation without SIFT
        SalesData.Reset();
        SalesData.SetRange("Customer No.", CustomerNo);
        if SalesData.FindSet() then
            repeat
                TotalAmount += SalesData."Sales Amount";
            until SalesData.Next() = 0;
        exit(TotalAmount);
    end;
}

// Benchmark results table for tracking
table 50102 "SIFT Performance Benchmark"
{
    fields
    {
        field(1; "Test Date"; DateTime) { }
        field(2; "Customer No."; Code[20]) { }
        field(3; "Record Count"; Integer) { }
        field(4; "SIFT Duration (ms)"; Integer) { }
        field(5; "Manual Duration (ms)"; Integer) { }
        field(6; "Performance Ratio"; Decimal) { }
    }
    
    keys
    {
        key(PK; "Test Date", "Customer No.") { }
    }
}
```

## Common SIFT Configuration Mistakes (Anti-Patterns)

### Anti-Pattern Examples with Corrections

#### ❌ Anti-Pattern 1: Excessive SIFT Keys
```al
// BAD: Too many SIFT keys with MaintainSIFTIndex = true
table 50103 "Over-SIFT Example (Bad)"
{
    keys
    {
        key(PK; ID) { }
        key(SIFT1; Field1) { SumIndexFields = Amount; MaintainSIFTIndex = true; }
        key(SIFT2; Field2) { SumIndexFields = Amount; MaintainSIFTIndex = true; }
        key(SIFT3; Field3) { SumIndexFields = Amount; MaintainSIFTIndex = true; }
        key(SIFT4; Field4) { SumIndexFields = Amount; MaintainSIFTIndex = true; }
        key(SIFT5; Field5) { SumIndexFields = Amount; MaintainSIFTIndex = true; }
        // Problem: Every data modification updates 5 SIFT indexes
        // Impact: Severe write performance degradation
    }
}
```

#### ✅ Corrected Pattern 1: Selective SIFT Usage
```al
// GOOD: Strategic SIFT key selection
table 50104 "Optimized SIFT Example (Good)"
{
    keys
    {
        key(PK; ID) { }
        
        // Only critical aggregations maintain real-time SIFT
        key(CriticalSIFT; "Customer No.") 
        { 
            SumIndexFields = Amount; 
            MaintainSIFTIndex = true;  // Dashboard requirement
        }
        
        // Secondary aggregations use on-demand calculation
        key(ReportingSIFT; "Item No.") 
        { 
            SumIndexFields = Amount; 
            MaintainSIFTIndex = false;  // Monthly reporting only
        }
    }
}
```

#### ❌ Anti-Pattern 2: SIFT Without SumIndexFields
```al
// BAD: SIFT configuration without aggregation fields
table 50105 "Empty SIFT Example (Bad)"
{
    keys
    {
        key(PK; ID) { }
        key(BadSIFT; "Customer No.") 
        { 
            MaintainSIFTIndex = true;  // No SumIndexFields specified!
            // Problem: Wastes storage and processing with no benefit
        }
    }
}
```

#### ✅ Corrected Pattern 2: Proper SIFT Configuration
```al
// GOOD: SIFT with appropriate aggregation fields
table 50106 "Proper SIFT Example (Good)"
{
    keys
    {
        key(PK; ID) { }
        key(GoodSIFT; "Customer No.") 
        { 
            SumIndexFields = "Sales Amount", Quantity, "Profit Amount";
            MaintainSIFTIndex = true;
        }
    }
}
```

#### ❌ Anti-Pattern 3: Wrong MaintainSIFTIndex Setting
```al
// BAD: Real-time SIFT for batch processing scenario
codeunit 50102 "Batch Import with Bad SIFT"
{
    procedure ImportLargeDataset()
    var
        DataRecord: Record "Over-SIFT Example (Bad)";
        i: Integer;
    begin
        // Importing 100,000 records
        for i := 1 to 100000 do begin
            DataRecord.Init();
            DataRecord.ID := i;
            DataRecord.Amount := Random(1000);
            DataRecord.Insert();  // Each insert updates multiple SIFT indexes!
            // Performance impact: 10x slower than necessary
        end;
    end;
}
```

#### ✅ Corrected Pattern 3: Appropriate SIFT for Usage Pattern
```al
// GOOD: On-demand SIFT for batch scenarios
table 50107 "Batch-Optimized SIFT (Good)"
{
    keys
    {
        key(PK; ID) { }
        key(BatchSIFT; "Data Type") 
        { 
            SumIndexFields = Amount;
            MaintainSIFTIndex = false;  // Calculated after batch import
        }
    }
}

codeunit 50103 "Batch Import with Good SIFT"
{
    procedure ImportLargeDataset()
    var
        DataRecord: Record "Batch-Optimized SIFT (Good)";
        i: Integer;
    begin
        // Fast import without real-time SIFT maintenance
        for i := 1 to 100000 do begin
            DataRecord.Init();
            DataRecord.ID := i;
            DataRecord.Amount := Random(1000);
            DataRecord.Insert();  // No SIFT overhead during import
        end;
        
        // Calculate aggregations on-demand after import
        DataRecord.CalcSums(Amount);  // Uses SIFT for efficient calculation
        Message('Total imported amount: %1', DataRecord.Amount);
    end;
}
```

## Best Practice Decision Matrix

### SIFT Configuration Guidelines
```al
// Decision framework for SIFT configuration
codeunit 50104 "SIFT Decision Framework"
{
    // Use this pattern to determine optimal SIFT settings
    procedure RecommendSIFTSettings(ReadFrequency: Text; WriteFrequency: Text; DataVolume: Text): Text
    begin
        case true of
            (ReadFrequency = 'High') and (WriteFrequency = 'Low'):
                exit('MaintainSIFTIndex = true - Optimal for dashboards and real-time reporting');
                
            (ReadFrequency = 'Low') and (WriteFrequency = 'High'):
                exit('MaintainSIFTIndex = false - Optimal for transaction processing systems');
                
            (ReadFrequency = 'Medium') and (WriteFrequency = 'Medium'):
                exit('Test both settings and measure performance - Consider time-based switching');
                
            DataVolume = 'Very Large':
                exit('MaintainSIFTIndex = false - Avoid real-time maintenance on large datasets');
                
            else
                exit('Start with MaintainSIFTIndex = false and optimize based on actual usage patterns');
        end;
    end;
}

/*
SIFT Performance Guidelines:

1. Real-time SIFT (MaintainSIFTIndex = true):
   - Best for: Dashboards, frequently accessed reports, user interface calculations
   - Trade-off: Slower writes, faster reads
   - Limit: Maximum 3-5 real-time SIFT keys per table

2. On-demand SIFT (MaintainSIFTIndex = false):
   - Best for: Periodic reporting, batch processing, large data imports
   - Trade-off: Faster writes, acceptable read performance
   - Benefit: No maintenance overhead during data modifications

3. Monitoring and Optimization:
   - Measure actual performance in production-like environments
   - Monitor SIFT calculation times and data modification impact
   - Consider data growth patterns when choosing SIFT strategy
   - Review SIFT usage patterns quarterly and adjust as needed
*/
```