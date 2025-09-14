# API Page Source Table Key Requirements - AL Code Sample

## Basic Primary Key Configuration

```al
table 50130 "API Customer Data"
{
    Caption = 'API Customer Data';
    DataClassification = CustomerContent;
    
    fields
    {
        field(1; "No."; Code[20])
        {
            Caption = 'No.';
            DataClassification = CustomerContent;
        }
        field(2; SystemId; Guid)
        {
            Caption = 'System ID';
            DataClassification = SystemMetadata;
        }
        field(10; Name; Text[100])
        {
            Caption = 'Name';
            DataClassification = CustomerContent;
        }
        field(20; "Credit Limit"; Decimal)
        {
            Caption = 'Credit Limit';
            DataClassification = CustomerContent;
        }
        field(30; "Last Modified DateTime"; DateTime)
        {
            Caption = 'Last Modified Date Time';
            DataClassification = SystemMetadata;
        }
    }
    
    keys
    {
        // Primary key - MUST be the first key for API performance
        key(PK; "No.")
        {
            Clustered = true;  // Clustered index for optimal performance
        }
        
        // SystemId key - REQUIRED for OData entity identification
        key(SystemIdKey; SystemId)
        {
            // SystemId must be unique and indexed for API operations
            Unique = true;
        }
        
        // Additional keys for API query performance
        key(NameKey; Name)
        {
            // Enable efficient filtering and sorting by name
        }
        
        key(LastModifiedKey; "Last Modified DateTime")
        {
            // Support delta sync and change tracking queries
        }
    }
}

// Corresponding API page with proper key utilization
page 50130 "Customer Data API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'customers';
    APIVersion = 'v1.0';
    EntityName = 'customerData';
    EntitySetName = 'customerData';
    SourceTable = "API Customer Data";
    DelayedInsert = true;
    
    layout
    {
        area(Content)
        {
            repeater(Records)
            {
                // SystemId field - leverages SystemId key
                field(id; Rec.SystemId)
                {
                    Caption = 'Id';
                    Editable = false;
                }
                
                // Primary business key - leverages primary key
                field(number; Rec."No.")
                {
                    Caption = 'Number';
                }
                
                field(name; Rec.Name)
                {
                    Caption = 'Name';
                }
                
                field(creditLimit; Rec."Credit Limit")
                {
                    Caption = 'Credit Limit';
                }
                
                field(lastModifiedDateTime; Rec."Last Modified DateTime")
                {
                    Caption = 'Last Modified Date Time';
                    Editable = false;
                }
            }
        }
    }
}
```

## Composite Key Configuration for API Tables

```al
table 50131 "API Sales Line Data"
{
    Caption = 'API Sales Line Data';
    DataClassification = CustomerContent;
    
    fields
    {
        field(1; "Document Type"; Enum "Sales Document Type")
        {
            Caption = 'Document Type';
            DataClassification = CustomerContent;
        }
        field(2; "Document No."; Code[20])
        {
            Caption = 'Document No.';
            DataClassification = CustomerContent;
        }
        field(3; "Line No."; Integer)
        {
            Caption = 'Line No.';
            DataClassification = CustomerContent;
        }
        field(4; SystemId; Guid)
        {
            Caption = 'System ID';
            DataClassification = SystemMetadata;
        }
        field(10; "Item No."; Code[20])
        {
            Caption = 'Item No.';
            DataClassification = CustomerContent;
        }
        field(20; Quantity; Decimal)
        {
            Caption = 'Quantity';
            DataClassification = CustomerContent;
        }
        field(30; "Unit Price"; Decimal)
        {
            Caption = 'Unit Price';
            DataClassification = CustomerContent;
        }
    }
    
    keys
    {
        // Composite primary key - Essential for line-based entities
        key(PK; "Document Type", "Document No.", "Line No.")
        {
            Clustered = true;  // Clustered for optimal range queries
        }
        
        // SystemId key - Required for OData entity operations
        key(SystemIdKey; SystemId)
        {
            Unique = true;
        }
        
        // Document header key - Efficient parent-child queries
        key(DocumentKey; "Document Type", "Document No.")
        {
            // Enables efficient filtering by document
            // Supports OData navigation properties
        }
        
        // Item analysis key - Business intelligence queries
        key(ItemKey; "Item No.", "Document Type")
        {
            // Enables item-based reporting and analytics
        }
        
        // Performance key for aggregations
        key(AggregationKey; "Document Type", "Item No.")
        {
            SumIndexFields = Quantity, "Unit Price";
            // SIFT key for efficient aggregation queries
        }
    }
}

// API page utilizing composite keys
page 50131 "Sales Line Data API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'sales';
    APIVersion = 'v1.0';
    EntityName = 'salesLineData';
    EntitySetName = 'salesLineData';
    SourceTable = "API Sales Line Data";
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
                    Editable = false;
                }
                
                // Primary key components
                field(documentType; Rec."Document Type")
                {
                    Caption = 'Document Type';
                }
                
                field(documentNumber; Rec."Document No.")
                {
                    Caption = 'Document Number';
                }
                
                field(lineNumber; Rec."Line No.")
                {
                    Caption = 'Line Number';
                }
                
                field(itemNumber; Rec."Item No.")
                {
                    Caption = 'Item Number';
                }
                
                field(quantity; Rec.Quantity)
                {
                    Caption = 'Quantity';
                }
                
                field(unitPrice; Rec."Unit Price")
                {
                    Caption = 'Unit Price';
                }
            }
        }
    }
    
    // Demonstrate key usage in triggers
    trigger OnNewRecord(BelowxRec: Boolean)
    begin
        // Auto-generate line number using primary key
        if Rec."Line No." = 0 then
            Rec."Line No." := GetNextLineNo(Rec."Document Type", Rec."Document No.");
    end;
    
    local procedure GetNextLineNo(DocType: Enum "Sales Document Type"; DocNo: Code[20]): Integer
    var
        SalesLine: Record "API Sales Line Data";
        LineNo: Integer;
    begin
        // Use primary key for efficient max line number query
        SalesLine.SetRange("Document Type", DocType);
        SalesLine.SetRange("Document No.", DocNo);
        SalesLine.SetCurrentKey("Document Type", "Document No.", "Line No.");  // Use primary key
        SalesLine.Ascending(false);  // Descending order to get highest line number
        
        if SalesLine.FindFirst() then
            LineNo := SalesLine."Line No." + 10000
        else
            LineNo := 10000;
            
        exit(LineNo);
    end;
}
```

## Key Performance Optimization Examples

```al
table 50132 "API Performance Demo"
{
    Caption = 'API Performance Demo';
    DataClassification = CustomerContent;
    
    fields
    {
        field(1; "Entry No."; Integer)
        {
            Caption = 'Entry No.';
            DataClassification = CustomerContent;
            AutoIncrement = true;  // Auto-increment primary key
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
        field(20; "Posting Date"; Date)
        {
            Caption = 'Posting Date';
            DataClassification = CustomerContent;
        }
        field(30; Amount; Decimal)
        {
            Caption = 'Amount';
            DataClassification = CustomerContent;
        }
        field(40; "Document No."; Code[20])
        {
            Caption = 'Document No.';
            DataClassification = CustomerContent;
        }
        field(50; "External Document No."; Code[35])
        {
            Caption = 'External Document No.';
            DataClassification = CustomerContent;
        }
    }
    
    keys
    {
        // Primary key - Sequential integer for optimal insert performance
        key(PK; "Entry No.")
        {
            Clustered = true;
        }
        
        // SystemId key - Required for API
        key(SystemIdKey; SystemId)
        {
            Unique = true;
        }
        
        // Business query keys - Designed for common API filter patterns
        key(CustomerDateKey; "Customer No.", "Posting Date")
        {
            // Optimal for: GET /api/entries?$filter=customerNo eq 'C001' and postingDate ge 2024-01-01
            // Includes Amount for covering index benefits
            IncludedFields = Amount, "Document No.";
        }
        
        key(DateCustomerKey; "Posting Date", "Customer No.")
        {
            // Optimal for: GET /api/entries?$filter=postingDate ge 2024-01-01&$orderby=customerNo
        }
        
        key(DocumentKey; "Document No.")
        {
            // Optimal for: GET /api/entries?$filter=documentNo eq 'INV-001'
        }
        
        key(ExternalDocKey; "External Document No.")
        {
            // Optimal for external system integration queries
        }
        
        // SIFT key for aggregation queries
        key(SIFTKey; "Customer No.", "Posting Date")
        {
            SumIndexFields = Amount;
            // Optimal for: GET /api/entries?$apply=groupby((customerNo),aggregate(amount with sum as total))
            MaintainSiftIndex = true;  // Real-time aggregations
        }
    }
}

// Performance-optimized API page
page 50132 "Performance Demo API"
{
    PageType = API;
    APIPublisher = 'contoso';
    APIGroup = 'performance';
    APIVersion = 'v1.0';
    EntityName = 'performanceEntry';
    EntitySetName = 'performanceEntries';
    SourceTable = "API Performance Demo";
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
                    Editable = false;  // Auto-increment field
                }
                
                field(customerNumber; Rec."Customer No.")
                {
                    Caption = 'Customer Number';
                }
                
                field(postingDate; Rec."Posting Date")
                {
                    Caption = 'Posting Date';
                }
                
                field(amount; Rec.Amount)
                {
                    Caption = 'Amount';
                }
                
                field(documentNumber; Rec."Document No.")
                {
                    Caption = 'Document Number';
                }
                
                field(externalDocumentNumber; Rec."External Document No.")
                {
                    Caption = 'External Document Number';
                }
            }
        }
    }
    
    // Demonstrate efficient key usage in code
    trigger OnOpenPage()
    begin
        // Set optimal key for typical API usage patterns
        SetOptimalKey();
    end;
    
    local procedure SetOptimalKey()
    begin
        // Default to customer-date key for most common queries
        Rec.SetCurrentKey("Customer No.", "Posting Date");
        
        // This key choice optimizes for common OData queries like:
        // $filter=customerNo eq 'value'
        // $filter=customerNo eq 'value' and postingDate ge datetime'2024-01-01T00:00:00Z'
        // $orderby=customerNo,postingDate
    end;
}

// Demonstrate key selection based on query patterns
codeunit 50132 "API Key Selection Demo"
{
    procedure DemonstrateKeyUsage()
    var
        PerfEntry: Record "API Performance Demo";
    begin
        // Example 1: Customer-based queries - Use CustomerDateKey
        PerfEntry.Reset();
        PerfEntry.SetCurrentKey("Customer No.", "Posting Date");  // Explicit key selection
        PerfEntry.SetRange("Customer No.", 'C001');
        PerfEntry.SetRange("Posting Date", CalcDate('<-1M>', Today), Today);
        // This query will use CustomerDateKey index efficiently
        
        // Example 2: Date range queries - Use DateCustomerKey  
        PerfEntry.Reset();
        PerfEntry.SetCurrentKey("Posting Date", "Customer No.");
        PerfEntry.SetRange("Posting Date", CalcDate('<-1W>', Today), Today);
        // This query will use DateCustomerKey index efficiently
        
        // Example 3: Document lookup - Use DocumentKey
        PerfEntry.Reset();
        PerfEntry.SetCurrentKey("Document No.");
        PerfEntry.SetRange("Document No.", 'INV-12345');
        // This query will use DocumentKey index efficiently
        
        // Example 4: External integration - Use ExternalDocKey
        PerfEntry.Reset();
        PerfEntry.SetCurrentKey("External Document No.");
        PerfEntry.SetRange("External Document No.", 'EXT-789');
        // This query will use ExternalDocKey index efficiently
    end;
    
    procedure DemonstrateSIFTUsage()
    var
        PerfEntry: Record "API Performance Demo";
        TotalAmount: Decimal;
    begin
        // SIFT aggregation using optimized key
        PerfEntry.Reset();
        PerfEntry.SetCurrentKey("Customer No.", "Posting Date");  // SIFT key
        PerfEntry.SetRange("Customer No.", 'C001');
        PerfEntry.SetRange("Posting Date", CalcDate('<-CY>', Today), Today);
        
        // This leverages the SIFT index for fast aggregation
        PerfEntry.CalcSums(Amount);
        TotalAmount := PerfEntry.Amount;
        
        // Equivalent OData query:
        // GET /api/performanceEntries?$filter=customerNo eq 'C001' and postingDate ge 2024-01-01
        //     &$apply=aggregate(amount with sum as total)
    end;
}
```

## Key Design Patterns for Different API Scenarios

```al
// Pattern 1: Master Data API Table Keys
table 50133 "API Item Master"
{
    fields
    {
        field(1; "No."; Code[20]) { }
        field(2; SystemId; Guid) { }
        field(10; Description; Text[100]) { }
        field(20; "Item Category Code"; Code[20]) { }
        field(30; "Unit Price"; Decimal) { }
        field(40; "Last Modified"; DateTime) { }
    }
    
    keys
    {
        // Primary key - Business identifier
        key(PK; "No.") { Clustered = true; }
        
        // SystemId - Required for OData
        key(SystemIdKey; SystemId) { Unique = true; }
        
        // Category grouping - Common filter
        key(CategoryKey; "Item Category Code", "No.") { }
        
        // Text search - Description lookups
        key(DescriptionKey; Description) { }
        
        // Sync/delta queries - Modified date
        key(ModifiedKey; "Last Modified", "No.") { }
    }
}

// Pattern 2: Transaction Data API Table Keys
table 50134 "API Transaction Log"
{
    fields
    {
        field(1; "Entry No."; BigInteger) { AutoIncrement = true; }
        field(2; SystemId; Guid) { }
        field(10; "Transaction Type"; Code[20]) { }
        field(20; "Source ID"; Code[50]) { }
        field(30; "Timestamp"; DateTime) { }
        field(40; "User ID"; Code[50]) { }
        field(50; Amount; Decimal) { }
    }
    
    keys
    {
        // Primary key - Sequential for performance
        key(PK; "Entry No.") { Clustered = true; }
        
        // SystemId - OData requirement
        key(SystemIdKey; SystemId) { Unique = true; }
        
        // Time-based queries - Most common pattern
        key(TimeKey; "Timestamp", "Transaction Type") 
        { 
            IncludedFields = "Source ID", Amount;  // Covering index
        }
        
        // Source tracking - Integration scenarios
        key(SourceKey; "Source ID", "Transaction Type", "Timestamp") { }
        
        // User activity - Audit queries
        key(UserKey; "User ID", "Timestamp") { }
        
        // Type-based aggregations - Reporting
        key(TypeAggKey; "Transaction Type", "Timestamp")
        {
            SumIndexFields = Amount;
            MaintainSiftIndex = false;  // On-demand for large tables
        }
    }
}

// Pattern 3: Hierarchical Data API Table Keys
table 50135 "API Organization Unit"
{
    fields
    {
        field(1; "Code"; Code[20]) { }
        field(2; SystemId; Guid) { }
        field(10; Name; Text[100]) { }
        field(20; "Parent Code"; Code[20]) { }
        field(30; Level; Integer) { }
        field(40; "Full Path"; Text[250]) { }
    }
    
    keys
    {
        // Primary key - Business code
        key(PK; "Code") { Clustered = true; }
        
        // SystemId - OData standard
        key(SystemIdKey; SystemId) { Unique = true; }
        
        // Hierarchy navigation - Parent-child
        key(ParentKey; "Parent Code", "Code") { }
        
        // Level-based queries - Reporting
        key(LevelKey; Level, "Code") { }
        
        // Path-based searches - Tree operations
        key(PathKey; "Full Path") { }
        
        // Name searches - User lookups
        key(NameKey; Name) { }
    }
}
```

## Implementation Notes

**Primary Key Design Principles:**
- Use business-meaningful keys when possible (No., Code)
- Consider sequential keys (AutoIncrement) for high-volume transaction data
- Ensure primary key supports typical query patterns
- Keep composite keys as narrow as possible for performance

**SystemId Key Requirements:**
- MUST be present and unique for all API tables
- Used by OData for entity identification and operations
- Automatically maintained by BC platform
- Required for proper ETag and concurrency control

**Query Optimization Keys:**
- Design keys based on common API filter patterns
- Use IncludedFields for covering indexes when beneficial
- Consider key order: most selective fields first
- Balance between query performance and maintenance overhead

**SIFT Key Considerations:**
- Use MaintainSiftIndex = true for frequently accessed aggregations
- Use MaintainSiftIndex = false for large tables with infrequent aggregations
- Include only necessary fields in SumIndexFields
- Monitor SIFT maintenance impact on write operations

**Performance Testing:**
- Test key effectiveness with realistic data volumes
- Monitor query execution plans in SQL Server
- Validate API response times meet requirements
- Consider storage overhead of additional indexes

**Key Maintenance:**
- Regularly review key usage statistics
- Remove unused keys to improve write performance
- Update key design as API usage patterns evolve
- Document key design decisions for team knowledge