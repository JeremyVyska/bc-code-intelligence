# OData Query Performance Optimization - AL Code Sample

## Basic OData Query Performance Patterns

```al
table 50140 "Sales Analytics Data"
{
    Caption = 'Sales Analytics Data';
    DataClassification = CustomerContent;
    
    fields
    {
        field(1; "Entry No."; BigInteger)
        {
            Caption = 'Entry No.';
            DataClassification = SystemMetadata;
            AutoIncrement = true;
        }
        field(2; SystemId; Guid)
        {
            Caption = 'System ID';
            DataClassification = SystemMetadata;
        }
        field(10; "Customer No."; Code[20])
        {
            Caption = 'Customer No.';
            DataClassification = CustomerContent;
        }
        field(20; "Item No."; Code[20])
        {
            Caption = 'Item No.';
            DataClassification = CustomerContent;
        }
        field(30; "Posting Date"; Date)
        {
            Caption = 'Posting Date';
            DataClassification = CustomerContent;
        }
        field(40; "Sales Amount"; Decimal)
        {
            Caption = 'Sales Amount';
            DataClassification = CustomerContent;
        }
        field(50; Quantity; Decimal)
        {
            Caption = 'Quantity';
            DataClassification = CustomerContent;
        }
        field(60; "Salesperson Code"; Code[20])
        {
            Caption = 'Salesperson Code';
            DataClassification = CustomerContent;
        }
        field(70; "Region Code"; Code[10])
        {
            Caption = 'Region Code';
            DataClassification = CustomerContent;
        }
    }
    
    keys
    {
        // Primary key - Sequential for optimal inserts
        key(PK; "Entry No.")
        {
            Clustered = true;
        }
        
        // SystemId key - Required for OData
        key(SystemIdKey; SystemId)
        {
            Unique = true;
        }
        
        // Optimized keys for common OData query patterns
        
        // Key 1: Customer date queries
        // Optimizes: $filter=customerNo eq 'C001' and postingDate ge 2024-01-01
        key(CustomerDateKey; "Customer No.", "Posting Date")
        {
            IncludedFields = "Sales Amount", Quantity, "Item No.";  // Covering index
        }
        
        // Key 2: Date range queries with various groupings
        // Optimizes: $filter=postingDate ge 2024-01-01&$orderby=postingDate,customerNo
        key(DateOptimizedKey; "Posting Date", "Customer No.", "Item No.")
        {
            IncludedFields = "Sales Amount", Quantity;
        }
        
        // Key 3: Item analysis queries
        // Optimizes: $filter=itemNo eq 'ITEM001'&$orderby=postingDate desc
        key(ItemAnalysisKey; "Item No.", "Posting Date")
        {
            IncludedFields = "Customer No.", "Sales Amount", Quantity;
        }
        
        // Key 4: Salesperson performance queries
        // Optimizes: $filter=salespersonCode eq 'SP001' and postingDate ge 2024-01-01
        key(SalespersonKey; "Salesperson Code", "Posting Date", "Customer No.")
        {
            IncludedFields = "Sales Amount", Quantity;
        }
        
        // Key 5: Regional analysis
        // Optimizes: $filter=regionCode eq 'WEST'&$apply=groupby((customerNo),aggregate(salesAmount with sum))
        key(RegionKey; "Region Code", "Customer No.", "Posting Date")
        {
            SumIndexFields = "Sales Amount", Quantity;  // SIFT for aggregations
            MaintainSiftIndex = true;
        }
    }
}

// Optimized API page with query hints
page 50140 "Sales Analytics API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'analytics';
    APIVersion = 'v2.0';
    EntityName = 'salesAnalytic';
    EntitySetName = 'salesAnalytics';
    SourceTable = "Sales Analytics Data";
    DelayedInsert = true;
    
    layout
    {
        area(Content)
        {
            repeater(Records)
            {
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                }
                
                field(entryNumber; Rec."Entry No.")
                {
                    Caption = 'Entry Number';
                    Editable = false;
                }
                
                field(customerNumber; Rec."Customer No.")
                {
                    Caption = 'Customer Number';
                }
                
                field(itemNumber; Rec."Item No.")
                {
                    Caption = 'Item Number';
                }
                
                field(postingDate; Rec."Posting Date")
                {
                    Caption = 'Posting Date';
                }
                
                field(salesAmount; Rec."Sales Amount")
                {
                    Caption = 'Sales Amount';
                }
                
                field(quantity; Rec.Quantity)
                {
                    Caption = 'Quantity';
                }
                
                field(salespersonCode; Rec."Salesperson Code")
                {
                    Caption = 'Salesperson Code';
                }
                
                field(regionCode; Rec."Region Code")
                {
                    Caption = 'Region Code';
                }
            }
        }
    }
    
    // Optimize query performance based on filter patterns
    trigger OnOpenPage()
    begin
        OptimizeForCommonQueries();
    end;
    
    local procedure OptimizeForCommonQueries()
    begin
        // Set default key for most common query pattern
        // This optimizes unfiltered requests and date-based queries
        Rec.SetCurrentKey("Posting Date", "Customer No.", "Item No.");
        
        // Enable read-ahead for batch processing
        Rec.SetAutoCalcFields();  // Only if FlowFields are present
    end;
}
```

## Advanced OData Query Optimization with Dynamic Key Selection

```al
codeunit 50140 "OData Query Optimizer"
{
    // Dynamic key selection based on OData query patterns
    
    procedure OptimizeQueryForFilters(var SalesAnalytics: Record "Sales Analytics Data"; FilterText: Text)
    begin
        // Analyze common OData filter patterns and set optimal keys
        
        if ContainsCustomerFilter(FilterText) then
            OptimizeForCustomerQueries(SalesAnalytics, FilterText)
        else if ContainsDateRangeFilter(FilterText) then
            OptimizeForDateQueries(SalesAnalytics)
        else if ContainsItemFilter(FilterText) then
            OptimizeForItemQueries(SalesAnalytics)
        else if ContainsSalespersonFilter(FilterText) then
            OptimizeForSalespersonQueries(SalesAnalytics)
        else
            OptimizeForGeneralQueries(SalesAnalytics);
    end;
    
    local procedure ContainsCustomerFilter(FilterText: Text): Boolean
    begin
        // Check if query contains customer number filters
        // Example: $filter=customerNo eq 'C001'
        exit(FilterText.Contains('customerNo') or FilterText.Contains('Customer'));
    end;
    
    local procedure ContainsDateRangeFilter(FilterText: Text): Boolean
    begin
        // Check if query contains date range filters
        // Example: $filter=postingDate ge 2024-01-01
        exit(FilterText.Contains('postingDate') or FilterText.Contains('Posting'));
    end;
    
    local procedure OptimizeForCustomerQueries(var SalesAnalytics: Record "Sales Analytics Data"; FilterText: Text)
    begin
        if ContainsDateRangeFilter(FilterText) then
            // Customer + date queries: use CustomerDateKey
            SalesAnalytics.SetCurrentKey("Customer No.", "Posting Date")
        else
            // Customer-only queries: still use CustomerDateKey for consistency
            SalesAnalytics.SetCurrentKey("Customer No.", "Posting Date");
    end;
    
    local procedure OptimizeForDateQueries(var SalesAnalytics: Record "Sales Analytics Data")
    begin
        // Date range queries: use DateOptimizedKey
        SalesAnalytics.SetCurrentKey("Posting Date", "Customer No.", "Item No.");
        
        // Enable ascending sort for date ranges (default behavior)
        SalesAnalytics.Ascending(true);
    end;
    
    local procedure OptimizeForItemQueries(var SalesAnalytics: Record "Sales Analytics Data")
    begin
        // Item-based queries: use ItemAnalysisKey
        SalesAnalytics.SetCurrentKey("Item No.", "Posting Date");
    end;
    
    local procedure OptimizeForSalespersonQueries(var SalesAnalytics: Record "Sales Analytics Data")
    begin
        // Salesperson queries: use SalespersonKey
        SalesAnalytics.SetCurrentKey("Salesperson Code", "Posting Date", "Customer No.");
    end;
    
    local procedure OptimizeForGeneralQueries(var SalesAnalytics: Record "Sales Analytics Data")
    begin
        // General queries: use date-optimized key as default
        SalesAnalytics.SetCurrentKey("Posting Date", "Customer No.", "Item No.");
    end;
}

// Performance monitoring and query analysis
codeunit 50141 "OData Performance Monitor"
{
    procedure LogQueryPerformance(QueryText: Text; ExecutionTime: Duration; RecordCount: Integer)
    var
        QueryLogEntry: Record "Query Performance Log";
    begin
        // Log query performance for analysis
        QueryLogEntry.Init();
        QueryLogEntry."Entry No." := GetNextEntryNo();
        QueryLogEntry."Query Text" := CopyStr(QueryText, 1, MaxStrLen(QueryLogEntry."Query Text"));
        QueryLogEntry."Execution Time (ms)" := ExecutionTime;
        QueryLogEntry."Record Count" := RecordCount;
        QueryLogEntry."Timestamp" := CurrentDateTime;
        QueryLogEntry."User ID" := UserId;
        QueryLogEntry.Insert();
        
        // Alert if query is slow
        if ExecutionTime > 5000 then  // 5 seconds
            LogSlowQueryAlert(QueryText, ExecutionTime);
    end;
    
    procedure AnalyzeQueryPattern(QueryText: Text): Text
    var
        Pattern: Text;
    begin
        // Analyze OData query patterns for optimization recommendations
        Pattern := 'UNKNOWN';
        
        if QueryText.Contains('$filter') then begin
            if QueryText.Contains('customerNo') then
                Pattern := 'CUSTOMER_FILTER';
            if QueryText.Contains('postingDate') then
                if Pattern <> 'UNKNOWN' then
                    Pattern += '+DATE_FILTER'
                else
                    Pattern := 'DATE_FILTER';
            if QueryText.Contains('itemNo') then
                if Pattern <> 'UNKNOWN' then
                    Pattern += '+ITEM_FILTER'
                else
                    Pattern := 'ITEM_FILTER';
        end;
        
        if QueryText.Contains('$orderby') then
            Pattern += '+SORT';
            
        if QueryText.Contains('$apply') then
            Pattern += '+AGGREGATION';
            
        if QueryText.Contains('$top') then
            Pattern += '+PAGING';
            
        exit(Pattern);
    end;
    
    procedure RecommendOptimization(QueryPattern: Text): Text
    var
        Recommendation: Text;
    begin
        // Provide optimization recommendations based on query patterns
        case QueryPattern of
            'CUSTOMER_FILTER':
                Recommendation := 'Use CustomerDateKey index. Consider SetCurrentKey("Customer No.", "Posting Date")';
            'DATE_FILTER':
                Recommendation := 'Use DateOptimizedKey index. Consider SetCurrentKey("Posting Date", "Customer No.", "Item No.")';
            'CUSTOMER_FILTER+DATE_FILTER':
                Recommendation := 'Optimal: CustomerDateKey handles this pattern efficiently';
            'ITEM_FILTER':
                Recommendation := 'Use ItemAnalysisKey index. Consider SetCurrentKey("Item No.", "Posting Date")';
            'CUSTOMER_FILTER+DATE_FILTER+AGGREGATION':
                Recommendation := 'Use RegionKey with SIFT. Consider CalcSums for aggregations';
            else
                Recommendation := 'Use default DateOptimizedKey for general queries';
        end;
        
        exit(Recommendation);
    end;
    
    local procedure LogSlowQueryAlert(QueryText: Text; ExecutionTime: Duration)
    begin
        // Log slow query alerts for DBA attention
        Message('Slow query detected: %1ms\nQuery: %2', ExecutionTime, QueryText);
    end;
}

// Supporting table for query performance logging
table 50141 "Query Performance Log"
{
    Caption = 'Query Performance Log';
    DataClassification = SystemMetadata;
    
    fields
    {
        field(1; "Entry No."; BigInteger)
        {
            Caption = 'Entry No.';
            AutoIncrement = true;
        }
        field(10; "Query Text"; Text[2048])
        {
            Caption = 'Query Text';
        }
        field(20; "Execution Time (ms)"; Duration)
        {
            Caption = 'Execution Time (ms)';
        }
        field(30; "Record Count"; Integer)
        {
            Caption = 'Record Count';
        }
        field(40; "Timestamp"; DateTime)
        {
            Caption = 'Timestamp';
        }
        field(50; "User ID"; Code[50])
        {
            Caption = 'User ID';
        }
    }
    
    keys
    {
        key(PK; "Entry No.")
        {
            Clustered = true;
        }
        key(TimeKey; "Timestamp")
        {
        }
    }
}
```

## OData Query Pattern Examples and Optimizations

```al
// Examples of OData queries and their AL optimizations
codeunit 50142 "OData Query Examples"
{
    procedure DemonstrateQueryOptimizations()
    var
        SalesAnalytics: Record "Sales Analytics Data";
    begin
        // Example 1: Simple customer filter
        // OData: GET /salesAnalytics?$filter=customerNo eq 'C001'
        DemoCustomerFilter(SalesAnalytics);
        
        // Example 2: Date range filter
        // OData: GET /salesAnalytics?$filter=postingDate ge 2024-01-01 and postingDate le 2024-12-31
        DemoDateRangeFilter(SalesAnalytics);
        
        // Example 3: Combined customer and date filter
        // OData: GET /salesAnalytics?$filter=customerNo eq 'C001' and postingDate ge 2024-01-01
        DemoCustomerDateFilter(SalesAnalytics);
        
        // Example 4: Sorting optimization
        // OData: GET /salesAnalytics?$orderby=postingDate desc,salesAmount desc
        DemoSortingOptimization(SalesAnalytics);
        
        // Example 5: Aggregation queries
        // OData: GET /salesAnalytics?$apply=groupby((customerNo),aggregate(salesAmount with sum as total))
        DemoAggregationQuery(SalesAnalytics);
        
        // Example 6: Paging optimization
        // OData: GET /salesAnalytics?$skip=100&$top=50
        DemoPagingOptimization(SalesAnalytics);
    end;
    
    local procedure DemoCustomerFilter(var SalesAnalytics: Record "Sales Analytics Data")
    begin
        // Optimized for: $filter=customerNo eq 'C001'
        SalesAnalytics.Reset();
        SalesAnalytics.SetCurrentKey("Customer No.", "Posting Date");  // Use CustomerDateKey
        SalesAnalytics.SetRange("Customer No.", 'C001');
        
        // This query will use the CustomerDateKey index efficiently
        // Index seek on Customer No. field, then range scan
        Message('Customer filter: %1 records found', SalesAnalytics.Count);
    end;
    
    local procedure DemoDateRangeFilter(var SalesAnalytics: Record "Sales Analytics Data")
    begin
        // Optimized for: $filter=postingDate ge 2024-01-01
        SalesAnalytics.Reset();
        SalesAnalytics.SetCurrentKey("Posting Date", "Customer No.", "Item No.");  // Use DateOptimizedKey
        SalesAnalytics.SetRange("Posting Date", 20240101D, 20241231D);
        
        // This query will use DateOptimizedKey for efficient range scan
        Message('Date range filter: %1 records found', SalesAnalytics.Count);
    end;
    
    local procedure DemoCustomerDateFilter(var SalesAnalytics: Record "Sales Analytics Data")
    begin
        // Optimized for: $filter=customerNo eq 'C001' and postingDate ge 2024-01-01
        SalesAnalytics.Reset();
        SalesAnalytics.SetCurrentKey("Customer No.", "Posting Date");  // Perfect key match
        SalesAnalytics.SetRange("Customer No.", 'C001');
        SalesAnalytics.SetRange("Posting Date", 20240101D, Today);
        
        // This query perfectly matches CustomerDateKey structure
        // Index seek on Customer No., then range scan on Posting Date
        Message('Customer + date filter: %1 records found', SalesAnalytics.Count);
    end;
    
    local procedure DemoSortingOptimization(var SalesAnalytics: Record "Sales Analytics Data")
    begin
        // Optimized for: $orderby=postingDate desc,salesAmount desc
        SalesAnalytics.Reset();
        SalesAnalytics.SetCurrentKey("Posting Date", "Customer No.", "Item No.");
        SalesAnalytics.Ascending(false);  // Descending order for posting date
        
        // Note: salesAmount sorting requires covering index or separate sort
        // Consider adding SalesAmount to IncludedFields if this pattern is common
        Message('Sorted query prepared (descending by date)');
    end;
    
    local procedure DemoAggregationQuery(var SalesAnalytics: Record "Sales Analytics Data")
    var
        TempSalesAnalytics: Record "Sales Analytics Data" temporary;
        CustomerNo: Code[20];
        TotalSales: Decimal;
    begin
        // Optimized for: $apply=groupby((customerNo),aggregate(salesAmount with sum as total))
        SalesAnalytics.Reset();
        SalesAnalytics.SetCurrentKey("Customer No.", "Posting Date");  // Use SIFT-enabled key
        
        // Use AL aggregation instead of individual record processing
        if SalesAnalytics.FindSet() then
            repeat
                if CustomerNo <> SalesAnalytics."Customer No." then begin
                    if CustomerNo <> '' then begin
                        // Output previous customer total
                        TempSalesAnalytics.Init();
                        TempSalesAnalytics."Customer No." := CustomerNo;
                        TempSalesAnalytics."Sales Amount" := TotalSales;
                        TempSalesAnalytics.Insert();
                    end;
                    CustomerNo := SalesAnalytics."Customer No.";
                    TotalSales := 0;
                end;
                TotalSales += SalesAnalytics."Sales Amount";
            until SalesAnalytics.Next() = 0;
            
        // Better approach: Use SIFT aggregation
        SalesAnalytics.Reset();
        SalesAnalytics.SetCurrentKey("Region Code", "Customer No.", "Posting Date");  // SIFT key
        SalesAnalytics.CalcSums("Sales Amount");
        Message('Total sales amount: %1', SalesAnalytics."Sales Amount");
    end;
    
    local procedure DemoPagingOptimization(var SalesAnalytics: Record "Sales Analytics Data")
    var
        PageSize: Integer;
        SkipCount: Integer;
        Counter: Integer;
    begin
        // Optimized for: $skip=100&$top=50
        PageSize := 50;
        SkipCount := 100;
        
        SalesAnalytics.Reset();
        SalesAnalytics.SetCurrentKey("Posting Date", "Customer No.", "Item No.");  // Stable sort order
        
        // Efficient paging using FindSet with skip logic
        if SalesAnalytics.FindSet() then begin
            // Skip records
            Counter := 0;
            while (Counter < SkipCount) and (SalesAnalytics.Next() <> 0) do
                Counter += 1;
                
            // Process page
            Counter := 0;
            repeat
                // Process record (in real API, this would be serialized to response)
                Counter += 1;
            until (Counter >= PageSize) or (SalesAnalytics.Next() = 0);
        end;
        
        Message('Page processed: %1 records', Counter);
    end;
}
```

## Performance Testing and Monitoring

```al
codeunit 50143 "OData Performance Testing"
{
    [Test]
    procedure TestCustomerFilterPerformance()
    var
        SalesAnalytics: Record "Sales Analytics Data";
        StartTime: DateTime;
        EndTime: DateTime;
        ExecutionTime: Duration;
    begin
        // Test performance of customer filter queries
        SalesAnalytics.Reset();
        SalesAnalytics.SetCurrentKey("Customer No.", "Posting Date");
        
        StartTime := CurrentDateTime;
        SalesAnalytics.SetRange("Customer No.", 'C001');
        SalesAnalytics.FindSet();
        EndTime := CurrentDateTime;
        
        ExecutionTime := EndTime - StartTime;
        
        // Assert performance is acceptable (< 1 second)
        Assert.IsTrue(ExecutionTime < 1000, 'Customer filter query too slow');
    end;
    
    [Test]
    procedure TestDateRangePerformance()
    var
        SalesAnalytics: Record "Sales Analytics Data";
        StartTime: DateTime;
        EndTime: DateTime;
        ExecutionTime: Duration;
    begin
        // Test performance of date range queries
        SalesAnalytics.Reset();
        SalesAnalytics.SetCurrentKey("Posting Date", "Customer No.", "Item No.");
        
        StartTime := CurrentDateTime;
        SalesAnalytics.SetRange("Posting Date", CalcDate('<-1Y>', Today), Today);
        SalesAnalytics.FindSet();
        EndTime := CurrentDateTime;
        
        ExecutionTime := EndTime - StartTime;
        
        // Assert performance is acceptable
        Assert.IsTrue(ExecutionTime < 2000, 'Date range query too slow');
    end;
    
    [Test]
    procedure TestAggregationPerformance()
    var
        SalesAnalytics: Record "Sales Analytics Data";
        StartTime: DateTime;
        EndTime: DateTime;
        ExecutionTime: Duration;
    begin
        // Test SIFT aggregation performance
        SalesAnalytics.Reset();
        SalesAnalytics.SetCurrentKey("Region Code", "Customer No.", "Posting Date");  // SIFT key
        
        StartTime := CurrentDateTime;
        SalesAnalytics.SetRange("Region Code", 'WEST');
        SalesAnalytics.CalcSums("Sales Amount", Quantity);
        EndTime := CurrentDateTime;
        
        ExecutionTime := EndTime - StartTime;
        
        // SIFT aggregations should be very fast
        Assert.IsTrue(ExecutionTime < 500, 'SIFT aggregation too slow');
    end;
}
```

## Implementation Notes

**Key Design for OData Performance:**
- Design keys based on common filter combinations in OData queries
- Use IncludedFields to create covering indexes for frequently accessed data
- Consider SIFT keys for aggregation queries ($apply operations)
- Ensure SystemId key is unique and indexed for entity operations

**Query Pattern Analysis:**
- Monitor actual OData queries to understand usage patterns
- Use query performance logging to identify slow queries
- Adjust key design based on real usage rather than assumptions
- Consider regional differences in query patterns

**Performance Optimization Techniques:**
- Set appropriate keys using SetCurrentKey before filtering
- Use efficient filtering with SetRange instead of SetFilter when possible
- Leverage SIFT aggregations for sum/count operations
- Implement proper paging to handle large result sets

**Monitoring and Maintenance:**
- Regularly analyze query performance logs
- Monitor index usage and effectiveness
- Update key design as API usage patterns evolve
- Test performance with realistic data volumes

**OData-Specific Considerations:**
- OData $filter translates to AL SetRange/SetFilter operations
- OData $orderby may require additional sorting if not supported by key
- OData $apply (aggregations) benefit significantly from SIFT indexes
- OData $expand (navigation properties) require efficient joins