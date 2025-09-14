# FlowField Optimization Examples

## Basic FlowField Optimization

### Efficient FlowField Configuration
```al
tableextension 50200 "Customer FlowField Optimization" extends Customer
{
    fields
    {
        // ✅ GOOD: Simple Sum FlowField with SIFT support
        field(50200; "Total Sales (Optimized)"; Decimal)
        {
            Caption = 'Total Sales (Optimized)';
            FieldClass = FlowField;
            CalcFormula = sum("Cust. Ledger Entry"."Sales (LCY)" WHERE("Customer No." = FIELD("No.")));
            Editable = false;
        }
        
        // ✅ GOOD: Count FlowField for record counting
        field(50201; "Invoice Count"; Integer)
        {
            Caption = 'Invoice Count';
            FieldClass = FlowField;
            CalcFormula = count("Sales Invoice Header" WHERE("Sell-to Customer No." = FIELD("No.")));
            Editable = false;
        }
        
        // ✅ GOOD: Lookup FlowField for direct access
        field(50202; "Primary Contact Name"; Text[100])
        {
            Caption = 'Primary Contact Name';
            FieldClass = FlowField;
            CalcFormula = lookup(Contact.Name WHERE("No." = FIELD("Primary Contact No.")));
            Editable = false;
        }
        
        // ✅ GOOD: Date-filtered FlowField with SIFT optimization
        field(50203; "Sales This Year (Filtered)"; Decimal)
        {
            Caption = 'Sales This Year';
            FieldClass = FlowField;
            CalcFormula = sum("Cust. Ledger Entry"."Sales (LCY)" WHERE("Customer No." = FIELD("No."), "Posting Date" = FIELD("Date Filter")));
            Editable = false;
        }
    }
}
```

## Strategic FlowField Calculation

### Efficient Page Implementation with FlowField Optimization
```al
page 50200 "Customer Performance Card"
{
    PageType = Card;
    SourceTable = Customer;
    
    layout
    {
        area(Content)
        {
            group(Statistics)
            {
                Caption = 'Performance Statistics';
                
                // ✅ IMMEDIATE: Essential fields calculated on record load
                field("Total Sales (Optimized)"; Rec."Total Sales (Optimized)")
                {
                    ApplicationArea = All;
                }
                field("Balance (LCY)"; Rec."Balance (LCY)")
                {
                    ApplicationArea = All;
                }
                
                // ✅ ON-DEMAND: Expensive fields calculated when accessed
                field("Detailed Entries"; DetailedEntriesCount)
                {
                    ApplicationArea = All;
                    trigger OnDrillDown()
                    begin
                        DetailedEntriesCount := GetDetailedEntriesCount();
                        ShowDetailedEntries();
                    end;
                }
            }
        }
    }
    
    var
        DetailedEntriesCount: Integer;
        FlowFieldsCalculated: Boolean;
        
    trigger OnAfterGetRecord()
    begin
        // ✅ EFFICIENT: Calculate essential FlowFields immediately
        Rec.CalcFields("Total Sales (Optimized)", "Balance (LCY)", "Invoice Count");
        
        // Reset expensive calculation flags
        Clear(DetailedEntriesCount);
        FlowFieldsCalculated := false;
    end;
    
    local procedure GetDetailedEntriesCount(): Integer
    var
        DetailedCustLedgEntry: Record "Detailed Cust. Ledg. Entry";
    begin
        if not FlowFieldsCalculated then begin
            // ✅ DEFERRED: Calculate expensive FlowField only when needed
            DetailedCustLedgEntry.SetRange("Customer No.", Rec."No.");
            DetailedEntriesCount := DetailedCustLedgEntry.Count();
            FlowFieldsCalculated := true;
        end;
        
        exit(DetailedEntriesCount);
    end;
}
```

## FlowField Performance Patterns

### Batch FlowField Calculation for Reports
```al
codeunit 50200 "FlowField Batch Calculator"
{
    procedure CalculateCustomerStatisticsBatch(var Customer: Record Customer)
    var
        TempCustomer: Record Customer temporary;
        StartTime: DateTime;
        Duration: Duration;
        ProcessedCount: Integer;
    begin
        StartTime := CurrentDateTime();
        
        // ✅ EFFICIENT: Collect customers for batch processing
        if Customer.FindSet() then begin
            repeat
                TempCustomer := Customer;
                TempCustomer.Insert();
                ProcessedCount += 1;
            until Customer.Next() = 0;
        end;
        
        // ✅ OPTIMIZED: Batch calculate FlowFields
        if TempCustomer.FindSet() then begin
            repeat
                Customer.Get(TempCustomer."No.");
                
                // Calculate multiple related FlowFields in single operation
                Customer.CalcFields(
                    "Total Sales (Optimized)", 
                    "Balance (LCY)", 
                    "Invoice Count",
                    "Sales This Year (Filtered)"
                );
                
                // Store calculated values if needed for further processing
                UpdateCustomerCache(Customer);
                
            until TempCustomer.Next() = 0;
        end;
        
        Duration := CurrentDateTime() - StartTime;
        Message('Calculated FlowFields for %1 customers in %2ms', ProcessedCount, Duration);
    end;
    
    local procedure UpdateCustomerCache(var Customer: Record Customer)
    var
        CustomerCache: Record "Customer Performance Cache";
    begin
        // ✅ CACHING: Store calculated values for reuse
        if not CustomerCache.Get(Customer."No.") then begin
            CustomerCache.Init();
            CustomerCache."Customer No." := Customer."No.";
            CustomerCache.Insert();
        end;
        
        CustomerCache."Total Sales" := Customer."Total Sales (Optimized)";
        CustomerCache."Current Balance" := Customer."Balance (LCY)";
        CustomerCache."Invoice Count" := Customer."Invoice Count";
        CustomerCache."Last Updated" := CurrentDateTime();
        CustomerCache.Modify();
    end;
}
```

### FlowField Caching Strategy
```al
table 50200 "Customer Performance Cache"
{
    Caption = 'Customer Performance Cache';
    DataClassification = CustomerContent;
    
    fields
    {
        field(1; "Customer No."; Code[20])
        {
            Caption = 'Customer No.';
            TableRelation = Customer;
        }
        field(10; "Total Sales"; Decimal)
        {
            Caption = 'Total Sales';
        }
        field(11; "Current Balance"; Decimal)
        {
            Caption = 'Current Balance';
        }
        field(12; "Invoice Count"; Integer)
        {
            Caption = 'Invoice Count';
        }
        field(20; "Last Updated"; DateTime)
        {
            Caption = 'Last Updated';
        }
        field(21; "Cache Valid Until"; DateTime)
        {
            Caption = 'Cache Valid Until';
        }
    }
    
    keys
    {
        key(PK; "Customer No.") { Clustered = true; }
        key(CacheValidity; "Cache Valid Until") { }
    }
}

codeunit 50201 "FlowField Cache Manager"
{
    procedure GetCachedCustomerStats(CustomerNo: Code[20]; var TotalSales: Decimal; var Balance: Decimal; var InvoiceCount: Integer): Boolean
    var
        CustomerCache: Record "Customer Performance Cache";
        Customer: Record Customer;
    begin
        // ✅ CACHE HIT: Return cached values if still valid
        if CustomerCache.Get(CustomerNo) then begin
            if CurrentDateTime() < CustomerCache."Cache Valid Until" then begin
                TotalSales := CustomerCache."Total Sales";
                Balance := CustomerCache."Current Balance";
                InvoiceCount := CustomerCache."Invoice Count";
                exit(true);
            end;
        end;
        
        // ✅ CACHE MISS: Calculate and cache values
        Customer.Get(CustomerNo);
        Customer.CalcFields("Total Sales (Optimized)", "Balance (LCY)", "Invoice Count");
        
        TotalSales := Customer."Total Sales (Optimized)";
        Balance := Customer."Balance (LCY)";
        InvoiceCount := Customer."Invoice Count";
        
        // Update cache with 5-minute validity
        UpdateCache(CustomerNo, TotalSales, Balance, InvoiceCount, CurrentDateTime() + 300000);
        
        exit(false); // Indicate cache miss
    end;
    
    local procedure UpdateCache(CustomerNo: Code[20]; TotalSales: Decimal; Balance: Decimal; InvoiceCount: Integer; ValidUntil: DateTime)
    var
        CustomerCache: Record "Customer Performance Cache";
    begin
        if not CustomerCache.Get(CustomerNo) then begin
            CustomerCache.Init();
            CustomerCache."Customer No." := CustomerNo;
            CustomerCache.Insert();
        end;
        
        CustomerCache."Total Sales" := TotalSales;
        CustomerCache."Current Balance" := Balance;
        CustomerCache."Invoice Count" := InvoiceCount;
        CustomerCache."Last Updated" := CurrentDateTime();
        CustomerCache."Cache Valid Until" := ValidUntil;
        CustomerCache.Modify();
    end;
    
    procedure InvalidateCustomerCache(CustomerNo: Code[20])
    var
        CustomerCache: Record "Customer Performance Cache";
    begin
        // ✅ INVALIDATION: Remove stale cache entries
        if CustomerCache.Get(CustomerNo) then begin
            CustomerCache."Cache Valid Until" := CurrentDateTime() - 1000; // Expire immediately
            CustomerCache.Modify();
        end;
    end;
    
    procedure CleanExpiredCache()
    var
        CustomerCache: Record "Customer Performance Cache";
    begin
        // ✅ CLEANUP: Remove expired cache entries
        CustomerCache.SetFilter("Cache Valid Until", '<%1', CurrentDateTime());
        CustomerCache.DeleteAll();
    end;
}
```

## Anti-Patterns to Avoid

### ❌ BAD: Inefficient FlowField Usage
```al
// ❌ BAD: Calculating FlowFields in loops
procedure CalculateCustomerTotals_BAD()
var
    Customer: Record Customer;
    TotalSales: Decimal;
begin
    if Customer.FindSet() then
        repeat
            // ❌ INEFFICIENT: Separate CalcFields call for each customer
            Customer.CalcFields("Sales (LCY)");
            TotalSales += Customer."Sales (LCY)";
            
            // ❌ WORSE: Multiple FlowField calculations per record
            Customer.CalcFields("Balance (LCY)");
            Customer.CalcFields("Net Change (LCY)");
            // Each CalcFields generates separate SQL queries
        until Customer.Next() = 0;
end;

// ❌ BAD: Complex FlowField without SIFT optimization
field(50299; "Complex Sales Calculation"; Decimal)
{
    FieldClass = FlowField;
    // ❌ BAD: Complex formula without supporting SIFT indexes
    CalcFormula = sum("Cust. Ledger Entry"."Sales (LCY)" WHERE(
        "Customer No." = FIELD("No."),
        "Posting Date" = FIELD("Date Filter"),
        "Document Type" = FILTER(Invoice|"Credit Memo"),
        "Salesperson Code" = FIELD("Salesperson Code"),
        "Global Dimension 1 Code" = FIELD("Global Dimension 1 Filter")
    ));
    // PROBLEM: No SIFT index supports this complex filter combination
}
```

## Advanced FlowField Optimization

### Conditional FlowField Calculation
```al
codeunit 50202 "Smart FlowField Calculator"
{
    procedure CalculateFlowFieldsConditionally(var Customer: Record Customer; RequiredFields: List of [Text])
    var
        FieldName: Text;
        StartTime: DateTime;
        Duration: Duration;
    begin
        StartTime := CurrentDateTime();
        
        // ✅ SMART: Calculate only requested FlowFields
        foreach FieldName in RequiredFields do begin
            case FieldName of
                'TotalSales':
                    Customer.CalcFields("Total Sales (Optimized)");
                'Balance':
                    Customer.CalcFields("Balance (LCY)");
                'InvoiceCount':
                    Customer.CalcFields("Invoice Count");
                'SalesThisYear':
                    begin
                        // Set date filter for year-to-date calculation
                        Customer.SetFilter("Date Filter", '%1..%2', DMY2Date(1, 1, Date2DMY(Today, 3)), Today);
                        Customer.CalcFields("Sales This Year (Filtered)");
                        Customer.SetRange("Date Filter");
                    end;
            end;
        end;
        
        Duration := CurrentDateTime() - StartTime;
        LogFlowFieldPerformance(Customer."No.", RequiredFields.Count(), Duration);
    end;
    
    local procedure LogFlowFieldPerformance(CustomerNo: Code[20]; FieldCount: Integer; Duration: Duration)
    var
        PerformanceLog: Record "FlowField Performance Log";
    begin
        PerformanceLog.Init();
        PerformanceLog."Customer No." := CustomerNo;
        PerformanceLog."Fields Calculated" := FieldCount;
        PerformanceLog."Calculation Duration" := Duration;
        PerformanceLog."Timestamp" := CurrentDateTime();
        PerformanceLog.Insert();
    end;
}
```

### FlowField Performance Monitoring
```al
table 50201 "FlowField Performance Log"
{
    fields
    {
        field(1; "Entry No."; Integer) { AutoIncrement = true; }
        field(2; "Customer No."; Code[20]) { }
        field(3; "Fields Calculated"; Integer) { }
        field(4; "Calculation Duration"; Duration) { }
        field(5; "Timestamp"; DateTime) { }
        field(6; "Cache Hit Rate"; Decimal) { }
    }
    
    keys
    {
        key(PK; "Entry No.") { Clustered = true; }
        key(Performance; "Customer No.", "Timestamp") { }
    }
}

codeunit 50203 "FlowField Performance Monitor"
{
    procedure AnalyzeFlowFieldPerformance(): JsonObject
    var
        PerformanceLog: Record "FlowField Performance Log";
        AnalysisResult: JsonObject;
        TotalDuration: Duration;
        Count: Integer;
        AverageDuration: Duration;
    begin
        // ✅ ANALYSIS: Calculate average FlowField performance
        PerformanceLog.SetFilter("Timestamp", '>=%1', CurrentDateTime() - 86400000); // Last 24 hours
        
        if PerformanceLog.FindSet() then begin
            repeat
                TotalDuration += PerformanceLog."Calculation Duration";
                Count += 1;
            until PerformanceLog.Next() = 0;
            
            if Count > 0 then
                AverageDuration := TotalDuration / Count;
        end;
        
        // Compile analysis results
        AnalysisResult.Add('average_duration_ms', AverageDuration);
        AnalysisResult.Add('total_calculations', Count);
        AnalysisResult.Add('analysis_period', '24 hours');
        AnalysisResult.Add('timestamp', CurrentDateTime());
        
        exit(AnalysisResult);
    end;
    
    procedure GetTopSlowCustomers(): List of [Code[20]]
    var
        PerformanceLog: Record "FlowField Performance Log";
        TempCustomerPerformance: Record "Integer" temporary;
        SlowCustomers: List of [Code[20]];
        CustomerNo: Code[20];
        TotalDuration: Duration;
        Count: Integer;
    begin
        // ✅ IDENTIFY: Find customers with slowest FlowField calculations
        PerformanceLog.SetFilter("Timestamp", '>=%1', CurrentDateTime() - 86400000);
        PerformanceLog.SetCurrentKey("Customer No.", "Timestamp");
        
        if PerformanceLog.FindSet() then begin
            repeat
                if CustomerNo <> PerformanceLog."Customer No." then begin
                    // Store previous customer's average
                    if (CustomerNo <> '') and (Count > 0) then
                        AddCustomerPerformance(CustomerNo, TotalDuration / Count, TempCustomerPerformance);
                    
                    // Start new customer
                    CustomerNo := PerformanceLog."Customer No.";
                    TotalDuration := 0;
                    Count := 0;
                end;
                
                TotalDuration += PerformanceLog."Calculation Duration";
                Count += 1;
            until PerformanceLog.Next() = 0;
            
            // Add final customer
            if (CustomerNo <> '') and (Count > 0) then
                AddCustomerPerformance(CustomerNo, TotalDuration / Count, TempCustomerPerformance);
        end;
        
        // Return top 10 slowest customers
        TempCustomerPerformance.SetCurrentKey("Integer");
        TempCustomerPerformance.Ascending(false);
        if TempCustomerPerformance.FindSet() then begin
            Count := 0;
            repeat
                SlowCustomers.Add(Format(TempCustomerPerformance."Integer"));
                Count += 1;
            until (TempCustomerPerformance.Next() = 0) or (Count >= 10);
        end;
        
        exit(SlowCustomers);
    end;
    
    local procedure AddCustomerPerformance(CustomerNo: Code[20]; AverageDuration: Duration; var TempCustomerPerformance: Record "Integer" temporary)
    begin
        TempCustomerPerformance.Init();
        TempCustomerPerformance.Number := CustomerNo; // Store customer in Number field
        TempCustomerPerformance."Integer" := AverageDuration; // Store duration for sorting
        TempCustomerPerformance.Insert();
    end;
}
```