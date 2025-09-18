# Named Parameter Pattern - AL Code Examples

## Good Examples

### Self-Documenting Variable Names
```al
procedure ProcessSalesOrder(DocumentNo: Code[20]): Boolean
var
    OrderToProcess: Code[20];
    ProcessingResult: Boolean;
    CustomerValidationPassed: Boolean;
    InventoryCheckSuccessful: Boolean;
begin
    OrderToProcess := DocumentNo;

    // Clear intent through descriptive variable names
    CustomerValidationPassed := ValidateCustomerCredit(OrderToProcess);
    if not CustomerValidationPassed then
        exit(false);

    InventoryCheckSuccessful := CheckInventoryAvailability(OrderToProcess);
    if not InventoryCheckSuccessful then
        exit(false);

    ProcessingResult := PostSalesOrder(OrderToProcess);
    exit(ProcessingResult);
end;
```

### Complex Parameter Clarity
```al
procedure CalculateItemCost(ItemNo: Code[20]; CalculationDate: Date; IncludeIndirectCosts: Boolean; UseFIFOMethod: Boolean): Decimal
var
    ItemNumberToCalculate: Code[20];
    CostCalculationDate: Date;
    ShouldIncludeIndirectCosts: Boolean;
    ShouldUseFIFOCosting: Boolean;
    CalculatedUnitCost: Decimal;
begin
    // Named variables make parameter usage crystal clear
    ItemNumberToCalculate := ItemNo;
    CostCalculationDate := CalculationDate;
    ShouldIncludeIndirectCosts := IncludeIndirectCosts;
    ShouldUseFIFOCosting := UseFIFOMethod;

    if ShouldUseFIFOCosting then
        CalculatedUnitCost := GetFIFOCost(ItemNumberToCalculate, CostCalculationDate)
    else
        CalculatedUnitCost := GetAverageCost(ItemNumberToCalculate, CostCalculationDate);

    if ShouldIncludeIndirectCosts then
        CalculatedUnitCost += GetIndirectCosts(ItemNumberToCalculate, CostCalculationDate);

    exit(CalculatedUnitCost);
end;
```

### Boolean Parameter Documentation
```al
procedure CreateCustomerStatement(CustomerNo: Code[20]; StartDate: Date; EndDate: Date; IncludeZeroBalances: Boolean; ShowDetailedEntries: Boolean; PrintImmediately: Boolean)
var
    CustomerToProcess: Code[20];
    StatementStartDate: Date;
    StatementEndDate: Date;
    ShouldIncludeZeroBalanceItems: Boolean;
    ShouldShowDetailedLedgerEntries: Boolean;
    ShouldPrintStatementImmediately: Boolean;
    StatementReport: Report "Customer Statement";
begin
    // Clear boolean intent through descriptive naming
    CustomerToProcess := CustomerNo;
    StatementStartDate := StartDate;
    StatementEndDate := EndDate;
    ShouldIncludeZeroBalanceItems := IncludeZeroBalances;
    ShouldShowDetailedLedgerEntries := ShowDetailedEntries;
    ShouldPrintStatementImmediately := PrintImmediately;

    StatementReport.SetTableView(CustomerToProcess, StatementStartDate, StatementEndDate);

    if ShouldIncludeZeroBalanceItems then
        StatementReport.SetIncludeZeroBalances();

    if ShouldShowDetailedLedgerEntries then
        StatementReport.SetShowDetails();

    if ShouldPrintStatementImmediately then
        StatementReport.Run()
    else
        StatementReport.RunModal();
end;
```

### API Integration with Clear Parameters
```al
procedure SyncWithExternalSystem(CustomerNo: Code[20]; SyncDirection: Option Import,Export,Bidirectional; ForceFullSync: Boolean; LogSyncActivity: Boolean): Boolean
var
    CustomerToSync: Code[20];
    ChosenSyncDirection: Option Import,Export,Bidirectional;
    ShouldForceCompleteSync: Boolean;
    ShouldLogAllActivity: Boolean;
    SyncResult: Boolean;
    ExternalAPIClient: Codeunit "External API Client";
begin
    // Parameter mapping with clear intent
    CustomerToSync := CustomerNo;
    ChosenSyncDirection := SyncDirection;
    ShouldForceCompleteSync := ForceFullSync;
    ShouldLogAllActivity := LogSyncActivity;

    if ShouldLogAllActivity then
        LogSyncStart(CustomerToSync, ChosenSyncDirection, ShouldForceCompleteSync);

    case ChosenSyncDirection of
        ChosenSyncDirection::Import:
            SyncResult := ExternalAPIClient.ImportCustomerData(CustomerToSync, ShouldForceCompleteSync);
        ChosenSyncDirection::Export:
            SyncResult := ExternalAPIClient.ExportCustomerData(CustomerToSync, ShouldForceCompleteSync);
        ChosenSyncDirection::Bidirectional:
            SyncResult := ExternalAPIClient.SyncCustomerBidirectional(CustomerToSync, ShouldForceCompleteSync);
    end;

    if ShouldLogAllActivity then
        LogSyncComplete(CustomerToSync, SyncResult);

    exit(SyncResult);
end;
```

## Bad Examples

### Unclear Parameter Usage
```al
procedure ProcessSalesOrder(DocumentNo: Code[20]): Boolean
var
    Rec: Record "Sales Header";
    Result: Boolean;
begin
    // BAD: Unclear what these boolean literals mean
    Result := ValidateOrder(DocumentNo, true, false, true);
    if not Result then
        exit(false);

    // BAD: Magic numbers and unclear parameters
    Result := CalculateCost(DocumentNo, 0, 1, true);
    if not Result then
        exit(false);

    // BAD: Multiple boolean parameters without context
    exit(PostOrder(DocumentNo, true, false, true, false));
end;
```

### Confusing Boolean Chains
```al
procedure CreateReport(CustomerNo: Code[20]; StartDate: Date; EndDate: Date): Boolean
begin
    // BAD: What do these boolean values represent?
    if GenerateCustomerReport(CustomerNo, StartDate, EndDate, true, false, true, false, true) then
        exit(ProcessReportOutput(CustomerNo, true, false))
    else
        exit(HandleReportError(CustomerNo, false, true));
end;
```

### Ambiguous Numeric Parameters
```al
procedure SetupItemPosting(ItemNo: Code[20]): Boolean
var
    Item: Record Item;
begin
    // BAD: What do these numbers mean?
    if ConfigureItemSettings(ItemNo, 1, 0, 2, 5) then begin
        // BAD: More unclear parameters
        exit(ApplyPostingSettings(ItemNo, 3, 1, 0));
    end;

    exit(false);
end;
```

## Best Practices

### Intermediate Variable Strategy
```al
procedure ProcessPurchaseOrder(DocumentNo: Code[20]; ValidateVendor: Boolean; CheckInventory: Boolean; PostImmediately: Boolean): Boolean
var
    OrderDocumentNumber: Code[20];
    ShouldValidateVendorCredit: Boolean;
    ShouldCheckInventoryLevels: Boolean;
    ShouldPostOrderImmediately: Boolean;
    ValidationSuccessful: Boolean;
    InventoryCheckPassed: Boolean;
    PostingSuccessful: Boolean;
begin
    // Map parameters to descriptive variables
    OrderDocumentNumber := DocumentNo;
    ShouldValidateVendorCredit := ValidateVendor;
    ShouldCheckInventoryLevels := CheckInventory;
    ShouldPostOrderImmediately := PostImmediately;

    // Clear conditional logic
    if ShouldValidateVendorCredit then begin
        ValidationSuccessful := ValidateVendorCredit(OrderDocumentNumber);
        if not ValidationSuccessful then
            exit(false);
    end;

    if ShouldCheckInventoryLevels then begin
        InventoryCheckPassed := CheckInventoryAvailability(OrderDocumentNumber);
        if not InventoryCheckPassed then
            exit(false);
    end;

    if ShouldPostOrderImmediately then begin
        PostingSuccessful := PostPurchaseOrder(OrderDocumentNumber);
        exit(PostingSuccessful);
    end;

    exit(true);
end;
```

### Option Parameter Clarity
```al
procedure GenerateFinancialReport(ReportType: Option Summary,Detailed,Comparative; OutputFormat: Option PDF,Excel,XML; EmailDelivery: Boolean): Boolean
var
    SelectedReportType: Option Summary,Detailed,Comparative;
    ChosenOutputFormat: Option PDF,Excel,XML;
    ShouldEmailReport: Boolean;
    ReportGenerator: Codeunit "Financial Report Generator";
    GenerationSuccessful: Boolean;
begin
    // Clear option handling
    SelectedReportType := ReportType;
    ChosenOutputFormat := OutputFormat;
    ShouldEmailReport := EmailDelivery;

    case SelectedReportType of
        SelectedReportType::Summary:
            GenerationSuccessful := ReportGenerator.GenerateSummaryReport(ChosenOutputFormat);
        SelectedReportType::Detailed:
            GenerationSuccessful := ReportGenerator.GenerateDetailedReport(ChosenOutputFormat);
        SelectedReportType::Comparative:
            GenerationSuccessful := ReportGenerator.GenerateComparativeReport(ChosenOutputFormat);
    end;

    if GenerationSuccessful and ShouldEmailReport then
        EmailReport(SelectedReportType, ChosenOutputFormat);

    exit(GenerationSuccessful);
end;
```

### Record Parameter Documentation
```al
procedure UpdateCustomerAddress(var CustomerRecord: Record Customer; NewAddressLine1: Text[100]; NewAddressLine2: Text[50]; NewCity: Text[30]; NewPostCode: Code[20]; ValidateAddress: Boolean): Boolean
var
    CustomerToUpdate: Record Customer;
    ProposedAddressLine1: Text[100];
    ProposedAddressLine2: Text[50];
    ProposedCity: Text[30];
    ProposedPostalCode: Code[20];
    ShouldValidateAddressWithPostalService: Boolean;
    AddressValidationSuccessful: Boolean;
begin
    // Clear parameter intent
    CustomerToUpdate := CustomerRecord;
    ProposedAddressLine1 := NewAddressLine1;
    ProposedAddressLine2 := NewAddressLine2;
    ProposedCity := NewCity;
    ProposedPostalCode := NewPostCode;
    ShouldValidateAddressWithPostalService := ValidateAddress;

    // Validate address if requested
    if ShouldValidateAddressWithPostalService then begin
        AddressValidationSuccessful := ValidateAddressWithPostalService(
            ProposedAddressLine1,
            ProposedAddressLine2,
            ProposedCity,
            ProposedPostalCode
        );

        if not AddressValidationSuccessful then
            exit(false);
    end;

    // Apply address changes
    CustomerToUpdate.Address := ProposedAddressLine1;
    CustomerToUpdate."Address 2" := ProposedAddressLine2;
    CustomerToUpdate.City := ProposedCity;
    CustomerToUpdate."Post Code" := ProposedPostalCode;
    CustomerToUpdate.Modify(true);

    // Return updated record
    CustomerRecord := CustomerToUpdate;
    exit(true);
end;
```

### Complex Calculation Clarity
```al
procedure CalculateExtendedPrice(BasePrice: Decimal; DiscountPercent: Decimal; TaxRate: Decimal; IncludeTax: Boolean; RoundToNearestCent: Boolean): Decimal
var
    ItemBasePrice: Decimal;
    ApplicableDiscountPercent: Decimal;
    ApplicableTaxRate: Decimal;
    ShouldIncludeTaxInCalculation: Boolean;
    ShouldRoundResultToNearestCent: Boolean;
    PriceAfterDiscount: Decimal;
    TaxAmount: Decimal;
    FinalCalculatedPrice: Decimal;
begin
    // Parameter mapping for clarity
    ItemBasePrice := BasePrice;
    ApplicableDiscountPercent := DiscountPercent;
    ApplicableTaxRate := TaxRate;
    ShouldIncludeTaxInCalculation := IncludeTax;
    ShouldRoundResultToNearestCent := RoundToNearestCent;

    // Calculate discount
    PriceAfterDiscount := ItemBasePrice * (1 - (ApplicableDiscountPercent / 100));

    // Calculate tax if required
    if ShouldIncludeTaxInCalculation then begin
        TaxAmount := PriceAfterDiscount * (ApplicableTaxRate / 100);
        FinalCalculatedPrice := PriceAfterDiscount + TaxAmount;
    end else begin
        FinalCalculatedPrice := PriceAfterDiscount;
    end;

    // Round if requested
    if ShouldRoundResultToNearestCent then
        FinalCalculatedPrice := Round(FinalCalculatedPrice, 0.01);

    exit(FinalCalculatedPrice);
end;
```

## Implementation Guidelines

1. **Use descriptive intermediate variables** - Map parameters to meaningful names immediately
2. **Apply consistent naming patterns** - Use prefixes like "Should", "Chosen", "Selected" for clarity
3. **Document boolean intent** - Make boolean parameter purpose clear through variable names
4. **Handle option parameters explicitly** - Use case statements with named option variables
5. **Group related parameters** - Organize parameter mapping logically in procedure start
6. **Validate parameter combinations** - Use clear variable names in validation logic