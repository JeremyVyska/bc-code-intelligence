# Variable Declaration Order - AL Code Examples

## Good Examples

### Logical Variable Declaration Order
```al
procedure ProcessSalesOrder(DocumentNo: Code[20]): Boolean
var
    // 1. Input validation and lookup records (in order of dependency)
    SalesHeader: Record "Sales Header";
    Customer: Record Customer;

    // 2. Processing records (in order of usage)
    SalesLine: Record "Sales Line";
    Item: Record Item;

    // 3. Temporary and working records
    TempBuffer: Record "Integer" temporary;

    // 4. Processing variables (grouped by type and purpose)
    ProcessingDate: Date;
    TotalAmount: Decimal;
    LineCount: Integer;
    i: Integer;

    // 5. Status and result variables
    IsProcessed: Boolean;
    HasErrors: Boolean;

    // 6. Error handling and messages (last)
    ErrorMessage: Text;
begin
    // Clear variable relationships and data flow
    // Easy to understand processing sequence
end;
```

### Complex Business Logic Organization
```al
procedure CalculateCustomerProfitability(CustomerNo: Code[20]; DateFilter: Text): Decimal
var
    // === PRIMARY ENTITY RECORDS ===
    Customer: Record Customer;

    // === FINANCIAL TRANSACTION RECORDS (in chronological dependency order) ===
    CustLedgerEntry: Record "Cust. Ledger Entry";
    DetailedCustLedgEntry: Record "Detailed Cust. Ledg. Entry";

    // === RELATED BUSINESS RECORDS ===
    SalesInvoiceHeader: Record "Sales Invoice Header";
    SalesInvoiceLine: Record "Sales Invoice Line";

    // === TEMPORARY CALCULATION RECORDS ===
    TempProfitabilityBuffer: Record "Integer" temporary;

    // === DATE AND PERIOD VARIABLES ===
    StartDate: Date;
    EndDate: Date;

    // === FINANCIAL CALCULATION VARIABLES (grouped by calculation type) ===
    // Revenue calculations
    TotalRevenue: Decimal;
    GrossRevenue: Decimal;
    NetRevenue: Decimal;

    // Cost calculations
    TotalCost: Decimal;
    DirectCost: Decimal;

    // Profit calculations
    GrossProfit: Decimal;
    NetProfit: Decimal;

    // === PROCESSING COUNTERS ===
    InvoiceCount: Integer;
    LineCount: Integer;

    // === CONTROL AND STATUS FLAGS ===
    HasData: Boolean;
    CalculationComplete: Boolean;

    // === ERROR HANDLING AND VALIDATION ===
    ValidationMessage: Text;
begin
    // Variable order reflects natural processing flow
end;
```

### Record Variables by Dependency Order
```al
procedure ProcessPurchaseOrder(): Boolean
var
    // Master data first (independent records)
    Vendor: Record Vendor;
    Item: Record Item;
    Location: Record Location;

    // Dependent records second
    PurchaseHeader: Record "Purchase Header";  // Depends on Vendor
    PurchaseLine: Record "Purchase Line";      // Depends on Header, Item, Location

    // Transaction records last
    PurchRcptHeader: Record "Purch. Rcpt. Header";
    PurchRcptLine: Record "Purch. Rcpt. Line";
begin
    // Process in dependency order
end;
```

## Bad Examples

### Random Variable Declaration Order
```al
procedure ProcessSalesOrder(DocumentNo: Code[20]): Boolean
var
    TotalAmount: Decimal;
    SalesLine: Record "Sales Line";
    i: Integer;
    Customer: Record Customer;
    IsProcessed: Boolean;
    SalesHeader: Record "Sales Header";
    TempBuffer: Record "Integer" temporary;
    ErrorMessage: Text;
    LineCount: Integer;
    HasErrors: Boolean;
    ProcessingDate: Date;
    Item: Record Item;
begin
    // Confusing variable order makes relationships unclear
    // Hard to understand data flow and dependencies
end;
```

### Mixed Types Without Grouping
```al
procedure BadOrganization(): Boolean
var
    Customer: Record Customer;
    TotalAmount: Decimal;
    SalesLine: Record "Sales Line";
    IsComplete: Boolean;
    Item: Record Item;
    Counter: Integer;
    TempRecord: Record "Integer" temporary;
    ErrorText: Text;
    ProcessingDate: Date;
    HasErrors: Boolean;
begin
    // No logical grouping or organization
    // Variables scattered without purpose
end;
```

## Best Practices

### Grouping by Purpose and Type
```al
procedure CalculateInventoryValue(): Decimal
var
    // Records first
    Item: Record Item;
    ItemLedgerEntry: Record "Item Ledger Entry";

    // Dates and options
    CalculationDate: Date;
    ValueMethod: Option FIFO,LIFO,Average;

    // Numeric calculations (from simple to complex)
    Quantity: Decimal;
    UnitCost: Decimal;
    TotalCost: Decimal;
    AverageUnitCost: Decimal;

    // Counters
    EntryCount: Integer;
    ProcessedCount: Integer;

    // Flags and status
    HasEntries: Boolean;
    CalculationSuccessful: Boolean;

    // Text and messages
    StatusMessage: Text;
begin
    // Clear progression from setup to results
end;
```

### Memory Access Optimization
```al
procedure ProcessLargeDataSet(): Boolean
var
    // Related counters grouped together
    TotalRecords: Integer;
    ProcessedRecords: Integer;
    ErrorRecords: Integer;
    SkippedRecords: Integer;

    // Related amounts grouped together
    TotalAmount: Decimal;
    ProcessedAmount: Decimal;
    RemainingAmount: Decimal;

    // Status flags grouped together
    HasErrors: Boolean;
    IsComplete: Boolean;
    ShouldContinue: Boolean;
begin
    // Better cache performance with grouped variables
end;
```

### Temporary Records Organization
```al
procedure BatchProcessItems(): Boolean
var
    // Source records
    Item: Record Item;

    // Temporary processing records (grouped by purpose)
    TempItemBuffer: Record "Item" temporary;
    TempProcessingLog: Record "Integer" temporary;
    TempErrorBuffer: Record "Error Message" temporary;

    // Processing variables
    BatchSize: Integer;
    ProcessedCount: Integer;

    // Results
    SuccessCount: Integer;
    ErrorCount: Integer;
begin
    // Logical flow from source through temporary processing to results
end;
```

## Implementation Guidelines

1. **Group by dependency** - Master records before dependent records
2. **Group by type** - Records, then simple types, then status variables
3. **Group by usage** - Variables used together should be declared together
4. **Use consistent patterns** - Apply same organization across all procedures
5. **Add section comments** - Use comments to separate logical groups
6. **Order by processing flow** - Declaration order should match usage order