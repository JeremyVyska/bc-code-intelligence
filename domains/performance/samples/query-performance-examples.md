# Query Performance Examples

## Efficient Filtering Patterns

### Index-Friendly Query Optimization
```al
codeunit 50300 "Query Performance Examples"
{
    // ✅ GOOD: Optimal filter sequence and index utilization
    procedure FindSalesOrdersOptimized(CustomerNo: Code[20]; DateFilter: Text; StatusFilter: Text): Boolean
    var
        SalesHeader: Record "Sales Header";
    begin
        // ✅ EFFICIENT: Most selective filters first
        SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order); // High selectivity
        SalesHeader.SetRange("Sell-to Customer No.", CustomerNo); // High selectivity
        SalesHeader.SetRange("Status", SalesHeader.Status::Open); // Medium selectivity
        
        if DateFilter <> '' then
            SalesHeader.SetFilter("Order Date", DateFilter); // Lower selectivity for ranges
        
        // ✅ KEY ALIGNMENT: Use key that matches filter sequence
        SalesHeader.SetCurrentKey("Document Type", "Sell-to Customer No.", "Status", "Order Date");
        
        exit(SalesHeader.FindSet());
    end;
    
    // ❌ BAD: Inefficient filter sequence
    procedure FindSalesOrdersInefficient(CustomerNo: Code[20]; DateFilter: Text): Boolean
    var
        SalesHeader: Record "Sales Header";
    begin
        // ❌ INEFFICIENT: Date filter first (low selectivity)
        if DateFilter <> '' then
            SalesHeader.SetFilter("Order Date", DateFilter);
        
        // ❌ INEFFICIENT: Document type filter last
        SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
        SalesHeader.SetRange("Sell-to Customer No.", CustomerNo);
        
        // ❌ BAD: Key doesn't match filter sequence
        exit(SalesHeader.FindSet());
    end;
}
```

## Record Traversal Optimization

### Efficient Record Processing Patterns
```al
codeunit 50301 "Record Traversal Optimization"
{
    // ✅ GOOD: Optimized record traversal with minimal field access
    procedure ProcessCustomersEfficiently()
    var
        Customer: Record Customer;
        TempProcessingQueue: Record "Integer" temporary;
        ProcessedCount: Integer;
        StartTime: DateTime;
    begin
        StartTime := CurrentDateTime();
        
        // ✅ PHASE 1: Identify candidates with minimal field loading
        Customer.SetLoadFields("No.", "Blocked", "Customer Posting Group");
        if Customer.FindSet() then
            repeat
                if ShouldProcessCustomer(Customer) then begin
                    TempProcessingQueue.Number := Customer."No.";
                    TempProcessingQueue.Insert();
                end;
            until Customer.Next() = 0;
        
        // ✅ PHASE 2: Process identified records with full field access
        ProcessedCount := ProcessIdentifiedCustomers(TempProcessingQueue);
        
        Message('Processed %1 customers in %2ms', ProcessedCount, CurrentDateTime() - StartTime);
    end;
    
    local procedure ShouldProcessCustomer(Customer: Record Customer): Boolean
    begin
        // ✅ EFFICIENT: Quick validation using loaded fields only
        exit((not Customer.Blocked) and (Customer."Customer Posting Group" <> ''));
    end;
    
    local procedure ProcessIdentifiedCustomers(var TempProcessingQueue: Record "Integer" temporary): Integer
    var
        Customer: Record Customer;
        ProcessedCount: Integer;
    begin
        // ✅ EFFICIENT: Full processing only for qualified records
        if TempProcessingQueue.FindSet() then
            repeat
                if Customer.Get(TempProcessingQueue.Number) then begin
                    ProcessSingleCustomer(Customer);
                    ProcessedCount += 1;
                end;
            until TempProcessingQueue.Next() = 0;
        
        exit(ProcessedCount);
    end;
    
    local procedure ProcessSingleCustomer(var Customer: Record Customer)
    begin
        // Full customer processing logic here
        Customer.CalcFields("Sales (LCY)", "Balance (LCY)");
        // Additional processing...
    end;
}
```

### Batch Query Processing
```al
codeunit 50302 "Batch Query Processor"
{
    procedure ProcessSalesLinesBatch(var SalesLineKeys: List of [Text])
    var
        SalesLine: Record "Sales Line";
        TempSalesLine: Record "Sales Line" temporary;
        DocumentKey: Text;
        KeyParts: List of [Text];
        DocumentType: Integer;
        DocumentNo: Code[20];
        LineNo: Integer;
    begin
        // ✅ COLLECT: Gather all lines in temporary table
        foreach DocumentKey in SalesLineKeys do begin
            KeyParts := DocumentKey.Split('|');
            if KeyParts.Count() = 3 then begin
                Evaluate(DocumentType, KeyParts.Get(1));
                DocumentNo := KeyParts.Get(2);
                Evaluate(LineNo, KeyParts.Get(3));
                
                if SalesLine.Get(DocumentType, DocumentNo, LineNo) then begin
                    TempSalesLine := SalesLine;
                    TempSalesLine.Insert();
                end;
            end;
        end;
        
        // ✅ PROCESS: Batch process collected records
        if TempSalesLine.FindSet() then
            repeat
                ProcessSalesLineWithCalculations(TempSalesLine);
            until TempSalesLine.Next() = 0;
    end;
    
    local procedure ProcessSalesLineWithCalculations(var SalesLine: Record "Sales Line")
    begin
        // ✅ EFFICIENT: All calculations in single context
        SalesLine.CalcFields("Reserved Quantity");
        SalesLine.Validate("Unit Price", GetOptimizedPrice(SalesLine));
        SalesLine.Modify();
    end;
    
    local procedure GetOptimizedPrice(SalesLine: Record "Sales Line"): Decimal
    var
        PriceCache: Dictionary of [Text, Decimal];
        CacheKey: Text;
        OptimizedPrice: Decimal;
    begin
        // ✅ CACHING: Avoid repeated price calculations
        CacheKey := StrSubstNo('%1|%2|%3', SalesLine."No.", SalesLine."Unit of Measure Code", SalesLine."Sell-to Customer No.");
        
        if PriceCache.ContainsKey(CacheKey) then
            exit(PriceCache.Get(CacheKey));
        
        // Calculate price and cache result
        OptimizedPrice := CalculateItemPrice(SalesLine);
        PriceCache.Set(CacheKey, OptimizedPrice);
        
        exit(OptimizedPrice);
    end;
    
    local procedure CalculateItemPrice(SalesLine: Record "Sales Line"): Decimal
    begin
        // Price calculation logic here
        exit(SalesLine."Unit Price");
    end;
}
```

## Advanced Query Patterns

### Join Optimization Strategies
```al
codeunit 50303 "Join Optimization Examples"
{
    // ✅ GOOD: Efficient join pattern with proper indexing
    procedure GetCustomerOrderSummary(CustomerNo: Code[20]): JsonObject
    var
        Customer: Record Customer;
        SalesHeader: Record "Sales Header";
        SalesLine: Record "Sales Line";
        SummaryJson: JsonObject;
        OrderArray: JsonArray;
        OrderJson: JsonObject;
        LineArray: JsonArray;
        TotalAmount: Decimal;
    begin
        // ✅ EFFICIENT: Start with most selective table
        if not Customer.Get(CustomerNo) then
            exit;
        
        // ✅ INDEXED JOIN: Use indexed fields for relationships
        SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
        SalesHeader.SetRange("Sell-to Customer No.", CustomerNo);
        SalesHeader.SetRange("Status", SalesHeader.Status::Open);
        SalesHeader.SetCurrentKey("Document Type", "Sell-to Customer No.", "Status");
        
        if SalesHeader.FindSet() then begin
            repeat
                Clear(OrderJson);
                OrderJson.Add('document_no', SalesHeader."No.");
                OrderJson.Add('order_date', SalesHeader."Order Date");
                
                // ✅ EFFICIENT: Nested query with proper key usage
                Clear(LineArray);
                SalesLine.SetRange("Document Type", SalesHeader."Document Type");
                SalesLine.SetRange("Document No.", SalesHeader."No.");
                SalesLine.SetCurrentKey("Document Type", "Document No.", "Line No.");
                
                if SalesLine.FindSet() then begin
                    repeat
                        AddLineToArray(SalesLine, LineArray);
                        TotalAmount += SalesLine."Amount Including VAT";
                    until SalesLine.Next() = 0;
                end;
                
                OrderJson.Add('lines', LineArray);
                OrderJson.Add('total_amount', TotalAmount);
                OrderArray.Add(OrderJson);
                
            until SalesHeader.Next() = 0;
        end;
        
        SummaryJson.Add('customer_no', CustomerNo);
        SummaryJson.Add('customer_name', Customer.Name);
        SummaryJson.Add('orders', OrderArray);
        SummaryJson.Add('grand_total', TotalAmount);
        
        exit(SummaryJson);
    end;
    
    local procedure AddLineToArray(SalesLine: Record "Sales Line"; var LineArray: JsonArray)
    var
        LineJson: JsonObject;
    begin
        LineJson.Add('line_no', SalesLine."Line No.");
        LineJson.Add('item_no', SalesLine."No.");
        LineJson.Add('description', SalesLine.Description);
        LineJson.Add('quantity', SalesLine.Quantity);
        LineJson.Add('unit_price', SalesLine."Unit Price");
        LineJson.Add('amount', SalesLine."Amount Including VAT");
        LineArray.Add(LineJson);
    end;
}
```

## Performance Measurement and Comparison

### Query Performance Testing Framework
```al
table 50300 "Query Performance Test Results"
{
    fields
    {
        field(1; "Test ID"; Code[20]) { }
        field(2; "Query Description"; Text[250]) { }
        field(3; "Execution Time (ms)"; Integer) { }
        field(4; "Records Processed"; Integer) { }
        field(5; "Index Used"; Boolean) { }
        field(6; "Test Timestamp"; DateTime) { }
        field(7; "Performance Rating"; Option) { OptionMembers = Excellent,Good,Fair,Poor; }
    }
    
    keys
    {
        key(PK; "Test ID", "Test Timestamp") { Clustered = true; }
        key(Performance; "Execution Time (ms)") { }
    }
}

codeunit 50304 "Query Performance Tester"
{
    procedure RunQueryPerformanceTest(TestID: Code[20]; Description: Text[250])
    var
        TestResults: Record "Query Performance Test Results";
        StartTime: DateTime;
        EndTime: DateTime;
        ExecutionTime: Integer;
        RecordsProcessed: Integer;
    begin
        StartTime := CurrentDateTime();
        
        // Execute the query being tested
        RecordsProcessed := ExecuteTestQuery(TestID);
        
        EndTime := CurrentDateTime();
        ExecutionTime := EndTime - StartTime;
        
        // Log performance results
        TestResults.Init();
        TestResults."Test ID" := TestID;
        TestResults."Query Description" := Description;
        TestResults."Execution Time (ms)" := ExecutionTime;
        TestResults."Records Processed" := RecordsProcessed;
        TestResults."Index Used" := CheckIndexUsage();
        TestResults."Test Timestamp" := CurrentDateTime();
        TestResults."Performance Rating" := CalculatePerformanceRating(ExecutionTime, RecordsProcessed);
        TestResults.Insert();
        
        DisplayTestResults(TestResults);
    end;
    
    local procedure ExecuteTestQuery(TestID: Code[20]): Integer
    var
        Customer: Record Customer;
        SalesHeader: Record "Sales Header";
        RecordCount: Integer;
    begin
        // ✅ SAMPLE QUERIES: Different test scenarios
        case TestID of
            'CUST_FILTER':
                begin
                    Customer.SetRange("Customer Posting Group", 'DOMESTIC');
                    Customer.SetRange("Blocked", false);
                    if Customer.FindSet() then
                        repeat
                            RecordCount += 1;
                        until Customer.Next() = 0;
                end;
            
            'SALES_JOIN':
                begin
                    SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
                    SalesHeader.SetRange("Status", SalesHeader.Status::Open);
                    if SalesHeader.FindSet() then
                        repeat
                            RecordCount += 1;
                            // Simulate join processing
                        until SalesHeader.Next() = 0;
                end;
        end;
        
        exit(RecordCount);
    end;
    
    local procedure CheckIndexUsage(): Boolean
    begin
        // ✅ INDEX VERIFICATION: Check if indexes were used efficiently
        // Implementation would analyze query execution plan
        // For this example, assume true
        exit(true);
    end;
    
    local procedure CalculatePerformanceRating(ExecutionTime: Integer; RecordsProcessed: Integer): Integer
    var
        PerformanceRating: Option Excellent,Good,Fair,Poor;
        TimePerRecord: Decimal;
    begin
        // ✅ RATING: Calculate performance rating based on metrics
        if RecordsProcessed > 0 then
            TimePerRecord := ExecutionTime / RecordsProcessed
        else
            TimePerRecord := ExecutionTime;
        
        case true of
            TimePerRecord < 0.1:
                exit(PerformanceRating::Excellent);
            TimePerRecord < 0.5:
                exit(PerformanceRating::Good);
            TimePerRecord < 2.0:
                exit(PerformanceRating::Fair);
            else
                exit(PerformanceRating::Poor);
        end;
    end;
    
    local procedure DisplayTestResults(TestResults: Record "Query Performance Test Results")
    begin
        Message('Query Test Results:\Test: %1\Description: %2\Execution Time: %3ms\Records: %4\Rating: %5',
                TestResults."Test ID",
                TestResults."Query Description",
                TestResults."Execution Time (ms)",
                TestResults."Records Processed",
                TestResults."Performance Rating");
    end;
    
    procedure CompareQueryMethods(Description: Text[250])
    var
        Method1Time, Method2Time: Integer;
        Method1Records, Method2Records: Integer;
        PerformanceImprovement: Decimal;
    begin
        // ✅ COMPARISON: Test different query approaches
        Message('Testing Query Method 1...');
        Method1Time := MeasureQueryMethod1(Method1Records);
        
        Message('Testing Query Method 2...');
        Method2Time := MeasureQueryMethod2(Method2Records);
        
        // Calculate improvement
        if Method2Time > 0 then
            PerformanceImprovement := (Method1Time - Method2Time) * 100.0 / Method2Time
        else
            PerformanceImprovement := 0;
        
        // Display comparison results
        Message('Query Performance Comparison:\%1\Method 1: %2ms (%3 records)\Method 2: %4ms (%5 records)\Improvement: %6%',
                Description,
                Method1Time, Method1Records,
                Method2Time, Method2Records,
                Round(PerformanceImprovement, 0.1));
    end;
    
    local procedure MeasureQueryMethod1(var RecordCount: Integer): Integer
    var
        Customer: Record Customer;
        StartTime: DateTime;
    begin
        StartTime := CurrentDateTime();
        
        // ✅ METHOD 1: Standard approach
        if Customer.FindSet() then
            repeat
                if (Customer."Customer Posting Group" = 'DOMESTIC') and (not Customer.Blocked) then
                    RecordCount += 1;
            until Customer.Next() = 0;
        
        exit(CurrentDateTime() - StartTime);
    end;
    
    local procedure MeasureQueryMethod2(var RecordCount: Integer): Integer
    var
        Customer: Record Customer;
        StartTime: DateTime;
    begin
        StartTime := CurrentDateTime();
        
        // ✅ METHOD 2: Optimized with filters
        Customer.SetRange("Customer Posting Group", 'DOMESTIC');
        Customer.SetRange("Blocked", false);
        RecordCount := Customer.Count();
        
        exit(CurrentDateTime() - StartTime);
    end;
}
```

## Query Optimization Anti-Patterns

### Common Query Performance Mistakes
```al
codeunit 50305 "Query Anti-Patterns"
{
    // ❌ BAD: Inefficient nested loops
    procedure CalculateCustomerTotals_BAD()
    var
        Customer: Record Customer;
        SalesHeader: Record "Sales Header";
        SalesLine: Record "Sales Line";
        CustomerTotal: Decimal;
    begin
        // ❌ INEFFICIENT: Nested loops without proper filtering
        if Customer.FindSet() then
            repeat
                CustomerTotal := 0;
                
                // ❌ BAD: Inner loop without proper key usage
                if SalesHeader.FindSet() then
                    repeat
                        if SalesHeader."Sell-to Customer No." = Customer."No." then begin
                            // ❌ WORSE: Triple nested loop
                            if SalesLine.FindSet() then
                                repeat
                                    if (SalesLine."Document Type" = SalesHeader."Document Type") and
                                       (SalesLine."Document No." = SalesHeader."No.") then
                                        CustomerTotal += SalesLine."Amount Including VAT";
                                until SalesLine.Next() = 0;
                        end;
                    until SalesHeader.Next() = 0;
                
                // Process customer total...
            until Customer.Next() = 0;
    end;
    
    // ✅ GOOD: Optimized approach with proper filtering
    procedure CalculateCustomerTotals_GOOD()
    var
        Customer: Record Customer;
        SalesHeader: Record "Sales Header";
        SalesLine: Record "Sales Line";
        CustomerTotal: Decimal;
    begin
        if Customer.FindSet() then
            repeat
                // ✅ EFFICIENT: Filtered query with proper key usage
                SalesHeader.SetRange("Sell-to Customer No.", Customer."No.");
                SalesHeader.SetCurrentKey("Sell-to Customer No.", "Document Type");
                
                CustomerTotal := 0;
                if SalesHeader.FindSet() then
                    repeat
                        // ✅ EFFICIENT: Use SIFT aggregation instead of line-by-line
                        SalesHeader.CalcFields("Amount Including VAT");
                        CustomerTotal += SalesHeader."Amount Including VAT";
                    until SalesHeader.Next() = 0;
                
                SalesHeader.SetRange("Sell-to Customer No.");
                
                // Process customer total...
            until Customer.Next() = 0;
    end;
}
```