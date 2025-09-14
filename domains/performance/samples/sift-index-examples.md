# SIFT Index Implementation Examples

## Basic SIFT Index Configuration

### Standard Sales Analytics Table with SIFT
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
        
        // ✅ GOOD: Customer-focused SIFT for dashboard performance
        key(CustomerSIFT; "Customer No.")
        {
            SumIndexFields = "Sales Amount", Quantity, "Profit Amount";
            MaintainSIFTIndex = true; // Real-time updates for instant access
        }
        
        // ✅ GOOD: Item analysis SIFT with selective fields
        key(ItemSIFT; "Item No.")
        {
            SumIndexFields = "Sales Amount", Quantity;
            MaintainSIFTIndex = true;
        }
        
        // ✅ GOOD: Date-based SIFT for reporting (on-demand)
        key(DateSIFT; "Posting Date")
        {
            SumIndexFields = "Sales Amount", "Profit Amount";
            MaintainSIFTIndex = false; // Calculated when needed for reports
        }
    }
}
```

## FlowField Integration with SIFT

### Customer Extension with SIFT-Optimized FlowFields
```al
tableextension 50100 "Customer SIFT Extension" extends Customer
{
    fields
    {
        // ✅ GOOD: FlowField leverages CustomerSIFT index
        field(50100; "Total Sales Amount"; Decimal)
        {
            Caption = 'Total Sales Amount';
            FieldClass = FlowField;
            CalcFormula = sum("Sales Analytics Data"."Sales Amount" WHERE("Customer No." = FIELD("No.")));
            Editable = false;
        }
        
        // ✅ GOOD: Quantity FlowField uses same SIFT index
        field(50101; "Total Quantity Sold"; Decimal)
        {
            Caption = 'Total Quantity Sold';
            FieldClass = FlowField;
            CalcFormula = sum("Sales Analytics Data".Quantity WHERE("Customer No." = FIELD("No.")));
            Editable = false;
        }
        
        // ✅ GOOD: Date-filtered FlowField with SIFT support
        field(50102; "Sales This Year"; Decimal)
        {
            Caption = 'Sales This Year';
            FieldClass = FlowField;
            CalcFormula = sum("Sales Analytics Data"."Sales Amount" WHERE("Customer No." = FIELD("No."), "Posting Date" = FIELD("Date Filter")));
            Editable = false;
        }
    }
}
```

## Performance Measurement Examples

### SIFT Performance Comparison
```al
codeunit 50100 "SIFT Performance Demo"
{
    procedure CompareSIFTPerformance(CustomerNo: Code[20])
    var
        Customer: Record Customer;
        SalesData: Record "Sales Analytics Data";
        StartTime: DateTime;
        SIFTDuration: Duration;
        ManualDuration: Duration;
        TotalAmount: Decimal;
    begin
        // Test 1: SIFT-optimized calculation
        StartTime := CurrentDateTime();
        TotalAmount := CalculateUsingSIFT(CustomerNo);
        SIFTDuration := CurrentDateTime() - StartTime;
        
        // Test 2: Manual calculation
        StartTime := CurrentDateTime();
        TotalAmount := CalculateManually(CustomerNo);
        ManualDuration := CurrentDateTime() - StartTime;
        
        // Display performance comparison
        Message('SIFT Calculation: %1ms\Manual Calculation: %2ms\Performance Improvement: %3x faster',
                SIFTDuration, ManualDuration, Round(ManualDuration / SIFTDuration, 0.1));
    end;
    
    local procedure CalculateUsingSIFT(CustomerNo: Code[20]): Decimal
    var
        Customer: Record Customer;
    begin
        // ✅ LEVERAGES SIFT: Uses CustomerSIFT index for O(1) lookup
        Customer.Get(CustomerNo);
        Customer.CalcFields("Total Sales Amount");
        exit(Customer."Total Sales Amount");
    end;
    
    local procedure CalculateManually(CustomerNo: Code[20]): Decimal
    var
        SalesData: Record "Sales Analytics Data";
        TotalAmount: Decimal;
    begin
        // ❌ SLOW: Manual calculation forces table scan
        SalesData.SetRange("Customer No.", CustomerNo);
        if SalesData.FindSet() then
            repeat
                TotalAmount += SalesData."Sales Amount";
            until SalesData.Next() = 0;
        exit(TotalAmount);
    end;
}
```

## Anti-Patterns to Avoid

### ❌ BAD: Excessive SIFT Configuration
```al
table 50101 "Over-SIFT Example (Bad)"
{
    keys
    {
        key(PK; ID) { Clustered = true; }
        
        // ❌ BAD: Too many SIFT indexes with real-time maintenance
        key(SIFT1; Field1) { SumIndexFields = Amount; MaintainSIFTIndex = true; }
        key(SIFT2; Field2) { SumIndexFields = Amount; MaintainSIFTIndex = true; }
        key(SIFT3; Field3) { SumIndexFields = Amount; MaintainSIFTIndex = true; }
        key(SIFT4; Field4) { SumIndexFields = Amount; MaintainSIFTIndex = true; }
        key(SIFT5; Field5) { SumIndexFields = Amount; MaintainSIFTIndex = true; }
        
        // PROBLEM: Every data modification updates 5 SIFT indexes
        // IMPACT: Write operations become 5x slower
    }
}
```

### ❌ BAD: SIFT Without Aggregation Fields
```al
table 50102 "Empty SIFT Example (Bad)"
{
    keys
    {
        key(PK; ID) { Clustered = true; }
        
        // ❌ BAD: MaintainSIFTIndex without SumIndexFields
        key(BadSIFT; "Customer No.") 
        { 
            MaintainSIFTIndex = true; // No benefit - no aggregation fields!
        }
    }
}
```

## Advanced SIFT Scenarios

### Multi-Dimensional SIFT for Analytics
```al
table 50103 "Advanced Sales Analysis"
{
    fields
    {
        field(1; "Salesperson Code"; Code[20]) { }
        field(2; "Customer No."; Code[20]) { }
        field(3; "Item Category"; Code[20]) { }
        field(4; "Posting Date"; Date) { }
        field(10; "Sales Amount"; Decimal) { }
        field(11; "Cost Amount"; Decimal) { }
        field(12; "Profit Amount"; Decimal) { }
    }
    
    keys
    {
        key(PK; "Salesperson Code", "Customer No.", "Item Category", "Posting Date") 
        { 
            Clustered = true; 
        }
        
        // ✅ GOOD: Primary SIFT for salesperson performance
        key(SalespersonSIFT; "Salesperson Code")
        {
            SumIndexFields = "Sales Amount", "Profit Amount";
            MaintainSIFTIndex = true; // Dashboard requirement
        }
        
        // ✅ GOOD: Category analysis SIFT (on-demand)
        key(CategorySIFT; "Item Category", "Posting Date")
        {
            SumIndexFields = "Sales Amount", "Cost Amount";
            MaintainSIFTIndex = false; // Monthly reports only
        }
    }
}
```

### SIFT Usage in Business Logic
```al
codeunit 50101 "Sales Performance Manager"
{
    procedure GetSalespersonRanking(): List of [Text]
    var
        SalesAnalysis: Record "Advanced Sales Analysis";
        TempRanking: Record "Name/Value Buffer" temporary;
        RankingList: List of [Text];
        SalespersonCode: Code[20];
    begin
        // ✅ EFFICIENT: Leverages SalespersonSIFT for rapid aggregation
        SalesAnalysis.SetCurrentKey("Salesperson Code");
        
        if SalesAnalysis.FindSet() then begin
            repeat
                if SalespersonCode <> SalesAnalysis."Salesperson Code" then begin
                    if SalespersonCode <> '' then
                        AddToRanking(SalespersonCode, SalesAnalysis, TempRanking);
                    
                    SalespersonCode := SalesAnalysis."Salesperson Code";
                    SalesAnalysis.SetRange("Salesperson Code", SalespersonCode);
                    SalesAnalysis.CalcSums("Sales Amount", "Profit Amount"); // Uses SIFT
                    SalesAnalysis.SetRange("Salesperson Code");
                end;
            until SalesAnalysis.Next() = 0;
            
            // Add final salesperson
            if SalespersonCode <> '' then
                AddToRanking(SalespersonCode, SalesAnalysis, TempRanking);
        end;
        
        // Sort and return ranking
        TempRanking.SetCurrentKey(Value);
        TempRanking.Ascending(false);
        if TempRanking.FindSet() then
            repeat
                RankingList.Add(StrSubstNo('%1: %2', TempRanking.Name, TempRanking.Value));
            until TempRanking.Next() = 0;
            
        exit(RankingList);
    end;
    
    local procedure AddToRanking(SalespersonCode: Code[20]; var SalesAnalysis: Record "Advanced Sales Analysis"; var TempRanking: Record "Name/Value Buffer" temporary)
    begin
        TempRanking.Init();
        TempRanking.Name := SalespersonCode;
        TempRanking.Value := Format(SalesAnalysis."Sales Amount");
        TempRanking.Insert();
    end;
}
```

## SIFT Maintenance Examples

### Batch Processing with SIFT Optimization
```al
codeunit 50102 "Optimized Batch Processor"
{
    procedure ProcessSalesDataBatch(var TempSalesData: Record "Sales Analytics Data" temporary)
    var
        SalesData: Record "Sales Analytics Data";
        ProcessedCount: Integer;
    begin
        // ✅ GOOD: Process in batches to minimize SIFT overhead
        if TempSalesData.FindSet() then begin
            repeat
                SalesData.TransferFields(TempSalesData);
                SalesData.Insert(); // Each insert updates SIFT indexes
                ProcessedCount += 1;
                
                // Commit every 1000 records to balance performance and recovery
                if ProcessedCount mod 1000 = 0 then
                    Commit();
                    
            until TempSalesData.Next() = 0;
            
            // Final commit
            if ProcessedCount mod 1000 <> 0 then
                Commit();
        end;
        
        Message('Processed %1 sales records with SIFT optimization', ProcessedCount);
    end;
}
```

## Performance Monitoring for SIFT

### SIFT Performance Tracking
```al
table 50104 "SIFT Performance Log"
{
    fields
    {
        field(1; "Entry No."; Integer) { AutoIncrement = true; }
        field(2; "Table Name"; Text[100]) { }
        field(3; "Operation Type"; Option) { OptionMembers = Insert,Modify,Delete,Calculate; }
        field(4; "Execution Time"; Duration) { }
        field(5; "Record Count"; Integer) { }
        field(6; "SIFT Indexes Updated"; Integer) { }
        field(7; "Timestamp"; DateTime) { }
    }
    
    keys
    {
        key(PK; "Entry No.") { Clustered = true; }
        key(Performance; "Table Name", "Operation Type", "Timestamp") { }
    }
}

codeunit 50103 "SIFT Performance Monitor"
{
    procedure LogSIFTOperation(TableName: Text[100]; OperationType: Option; Duration: Duration; RecordCount: Integer; SIFTCount: Integer)
    var
        PerformanceLog: Record "SIFT Performance Log";
    begin
        PerformanceLog.Init();
        PerformanceLog."Table Name" := TableName;
        PerformanceLog."Operation Type" := OperationType;
        PerformanceLog."Execution Time" := Duration;
        PerformanceLog."Record Count" := RecordCount;
        PerformanceLog."SIFT Indexes Updated" := SIFTCount;
        PerformanceLog."Timestamp" := CurrentDateTime();
        PerformanceLog.Insert();
    end;
    
    procedure GetAveragePerformance(TableName: Text[100]; OperationType: Option): Duration
    var
        PerformanceLog: Record "SIFT Performance Log";
        TotalDuration: Duration;
        Count: Integer;
    begin
        PerformanceLog.SetRange("Table Name", TableName);
        PerformanceLog.SetRange("Operation Type", OperationType);
        PerformanceLog.SetFilter("Timestamp", '>=%1', CurrentDateTime - 86400000); // Last 24 hours
        
        if PerformanceLog.FindSet() then begin
            repeat
                TotalDuration += PerformanceLog."Execution Time";
                Count += 1;
            until PerformanceLog.Next() = 0;
            
            if Count > 0 then
                exit(TotalDuration / Count);
        end;
        
        exit(0);
    end;
}
```