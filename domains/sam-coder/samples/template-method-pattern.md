# Template Method Pattern Sample

## Overview
This sample demonstrates template method pattern implementation in AL including algorithm structure definition, customizable steps, and inheritance-based customization.

## Basic Template Method Pattern

```al
codeunit 50500 "Document Processing Template"
{
    procedure ProcessDocument(var Document: Variant): Boolean
    begin
        if not ValidateDocument(Document) then
            exit(false);

        if not PreProcessDocument(Document) then
            exit(false);

        if not ExecuteMainProcessing(Document) then
            exit(false);

        if not PostProcessDocument(Document) then
            exit(false);

        if not FinalizeDocument(Document) then
            exit(false);

        exit(true);
    end;

    // Template methods (can be overridden by subclasses)
    protected procedure ValidateDocument(var Document: Variant): Boolean
    begin
        // Default validation logic
        exit(true);
    end;

    protected procedure PreProcessDocument(var Document: Variant): Boolean
    begin
        // Default pre-processing
        exit(true);
    end;

    // Abstract method (must be implemented by subclasses)
    protected procedure ExecuteMainProcessing(var Document: Variant): Boolean
    begin
        Error('ExecuteMainProcessing must be implemented by derived class');
    end;

    protected procedure PostProcessDocument(var Document: Variant): Boolean
    begin
        // Default post-processing
        exit(true);
    end;

    protected procedure FinalizeDocument(var Document: Variant): Boolean
    begin
        // Default finalization
        exit(true);
    end;
}
```

## Sales Order Processing Template

```al
codeunit 50501 "Sales Order Processing" extends "Document Processing Template"
{
    protected procedure ValidateDocument(var Document: Variant): Boolean
    var
        SalesHeader: Record "Sales Header";
    begin
        SalesHeader := Document;

        if SalesHeader."Sell-to Customer No." = '' then
            exit(false);

        if not CustomerExists(SalesHeader."Sell-to Customer No.") then
            exit(false);

        if SalesHeader."Order Date" = 0D then
            exit(false);

        exit(true);
    end;

    protected procedure PreProcessDocument(var Document: Variant): Boolean
    var
        SalesHeader: Record "Sales Header";
        SalesLine: Record "Sales Line";
    begin
        SalesHeader := Document;

        // Apply default values
        if SalesHeader."Posting Date" = 0D then
            SalesHeader."Posting Date" := WorkDate();

        // Initialize sales lines
        SalesLine.SetRange("Document Type", SalesHeader."Document Type");
        SalesLine.SetRange("Document No.", SalesHeader."No.");
        if SalesLine.FindSet() then
            repeat
                PreProcessSalesLine(SalesLine);
            until SalesLine.Next() = 0;

        Document := SalesHeader;
        exit(true);
    end;

    protected procedure ExecuteMainProcessing(var Document: Variant): Boolean
    var
        SalesHeader: Record "Sales Header";
        SalesPost: Codeunit "Sales-Post";
    begin
        SalesHeader := Document;

        // Execute the main sales processing
        SalesHeader.Ship := true;
        SalesHeader.Invoice := true;

        if not SalesPost.Run(SalesHeader) then
            exit(false);

        Document := SalesHeader;
        exit(true);
    end;

    protected procedure PostProcessDocument(var Document: Variant): Boolean
    var
        SalesHeader: Record "Sales Header";
    begin
        SalesHeader := Document;

        // Send confirmation email
        SendOrderConfirmation(SalesHeader);

        // Update customer statistics
        UpdateCustomerStatistics(SalesHeader);

        // Notify warehouse
        NotifyWarehouse(SalesHeader);

        Document := SalesHeader;
        exit(true);
    end;

    local procedure PreProcessSalesLine(var SalesLine: Record "Sales Line")
    begin
        // Calculate pricing
        if SalesLine."Unit Price" = 0 then
            SalesLine.Validate("Unit Price", GetItemPrice(SalesLine."No."));

        // Apply discounts
        ApplyLineDiscount(SalesLine);

        SalesLine.Modify();
    end;
}
```

## Purchase Order Processing Template

```al
codeunit 50502 "Purchase Order Processing" extends "Document Processing Template"
{
    protected procedure ValidateDocument(var Document: Variant): Boolean
    var
        PurchaseHeader: Record "Purchase Header";
    begin
        PurchaseHeader := Document;

        if PurchaseHeader."Buy-from Vendor No." = '' then
            exit(false);

        if not VendorExists(PurchaseHeader."Buy-from Vendor No.") then
            exit(false);

        if PurchaseHeader."Order Date" = 0D then
            exit(false);

        exit(true);
    end;

    protected procedure PreProcessDocument(var Document: Variant): Boolean
    var
        PurchaseHeader: Record "Purchase Header";
    begin
        PurchaseHeader := Document;

        // Set expected receipt date
        if PurchaseHeader."Expected Receipt Date" = 0D then
            PurchaseHeader."Expected Receipt Date" := CalcDate('+7D', WorkDate());

        // Apply vendor-specific defaults
        ApplyVendorDefaults(PurchaseHeader);

        Document := PurchaseHeader;
        exit(true);
    end;

    protected procedure ExecuteMainProcessing(var Document: Variant): Boolean
    var
        PurchaseHeader: Record "Purchase Header";
        PurchPost: Codeunit "Purch.-Post";
    begin
        PurchaseHeader := Document;

        // Execute the main purchase processing
        PurchaseHeader.Receive := true;
        PurchaseHeader.Invoice := true;

        if not PurchPost.Run(PurchaseHeader) then
            exit(false);

        Document := PurchaseHeader;
        exit(true);
    end;

    protected procedure PostProcessDocument(var Document: Variant): Boolean
    var
        PurchaseHeader: Record "Purchase Header";
    begin
        PurchaseHeader := Document;

        // Update inventory
        UpdateInventoryLevels(PurchaseHeader);

        // Send receipt confirmation
        SendReceiptConfirmation(PurchaseHeader);

        Document := PurchaseHeader;
        exit(true);
    end;
}
```

## Configurable Template Pattern

```al
codeunit 50503 "Configurable Process Template"
{
    var
        ProcessConfiguration: Record "Process Configuration";

    procedure ProcessWithConfiguration(var Document: Variant; ConfigCode: Code[20]): Boolean
    begin
        LoadConfiguration(ConfigCode);

        if ProcessConfiguration."Validate Documents" then
            if not ValidateDocument(Document) then
                exit(false);

        if ProcessConfiguration."Enable Pre Processing" then
            if not PreProcessDocument(Document) then
                exit(false);

        if not ExecuteMainProcessing(Document) then
            exit(false);

        if ProcessConfiguration."Enable Post Processing" then
            if not PostProcessDocument(Document) then
                exit(false);

        if ProcessConfiguration."Auto Finalize" then
            if not FinalizeDocument(Document) then
                exit(false);

        exit(true);
    end;

    protected procedure ExecuteMainProcessing(var Document: Variant): Boolean
    var
        ProcessingStrategy: Interface "Processing Strategy";
    begin
        ProcessingStrategy := GetProcessingStrategy();
        exit(ProcessingStrategy.Process(Document));
    end;

    local procedure LoadConfiguration(ConfigCode: Code[20])
    begin
        if not ProcessConfiguration.Get(ConfigCode) then
            Error('Process configuration %1 not found', ConfigCode);
    end;

    local procedure GetProcessingStrategy(): Interface "Processing Strategy"
    var
        StandardStrategy: Codeunit "Standard Processing Strategy";
        FastStrategy: Codeunit "Fast Processing Strategy";
        DetailedStrategy: Codeunit "Detailed Processing Strategy";
    begin
        case ProcessConfiguration."Processing Mode" of
            ProcessConfiguration."Processing Mode"::Standard:
                exit(StandardStrategy);
            ProcessConfiguration."Processing Mode"::Fast:
                exit(FastStrategy);
            ProcessConfiguration."Processing Mode"::Detailed:
                exit(DetailedStrategy);
        end;
    end;
}
```

## Data Import Template Pattern

```al
codeunit 50504 "Data Import Template"
{
    procedure ImportData(ImportSource: Text; ImportType: Enum "Import Type"): Boolean
    begin
        if not ValidateImportSource(ImportSource, ImportType) then
            exit(false);

        if not PrepareImport(ImportSource, ImportType) then
            exit(false);

        if not ParseData(ImportSource, ImportType) then
            exit(false);

        if not ValidateData(ImportType) then
            exit(false);

        if not TransformData(ImportType) then
            exit(false);

        if not LoadData(ImportType) then
            exit(false);

        if not FinalizeImport(ImportType) then
            exit(false);

        exit(true);
    end;

    // Template methods with default implementations
    protected procedure ValidateImportSource(ImportSource: Text; ImportType: Enum "Import Type"): Boolean
    begin
        exit(ImportSource <> '');
    end;

    protected procedure PrepareImport(ImportSource: Text; ImportType: Enum "Import Type"): Boolean
    begin
        // Default preparation logic
        exit(true);
    end;

    // Must be overridden for specific import types
    protected procedure ParseData(ImportSource: Text; ImportType: Enum "Import Type"): Boolean
    begin
        Error('ParseData must be implemented for import type %1', ImportType);
    end;

    protected procedure ValidateData(ImportType: Enum "Import Type"): Boolean
    begin
        exit(true);
    end;

    protected procedure TransformData(ImportType: Enum "Import Type"): Boolean
    begin
        exit(true);
    end;

    protected procedure LoadData(ImportType: Enum "Import Type"): Boolean
    begin
        Error('LoadData must be implemented for import type %1', ImportType);
    end;

    protected procedure FinalizeImport(ImportType: Enum "Import Type"): Boolean
    begin
        exit(true);
    end;
}
```

## CSV Import Implementation

```al
codeunit 50505 "CSV Customer Import" extends "Data Import Template"
{
    var
        TempCustomerBuffer: Record "Customer Import Buffer" temporary;

    protected procedure ParseData(ImportSource: Text; ImportType: Enum "Import Type"): Boolean
    var
        CSVReader: Codeunit "CSV Reader";
        DataLine: List of [Text];
    begin
        CSVReader.Initialize(ImportSource);

        while CSVReader.ReadLine(DataLine) do begin
            if not ParseCustomerLine(DataLine) then
                exit(false);
        end;

        exit(true);
    end;

    protected procedure ValidateData(ImportType: Enum "Import Type"): Boolean
    var
        ValidationErrors: Integer;
    begin
        ValidationErrors := 0;

        if TempCustomerBuffer.FindSet() then
            repeat
                if not ValidateCustomerData(TempCustomerBuffer) then
                    ValidationErrors += 1;
            until TempCustomerBuffer.Next() = 0;

        exit(ValidationErrors = 0);
    end;

    protected procedure TransformData(ImportType: Enum "Import Type"): Boolean
    begin
        if TempCustomerBuffer.FindSet() then
            repeat
                TransformCustomerData(TempCustomerBuffer);
                TempCustomerBuffer.Modify();
            until TempCustomerBuffer.Next() = 0;

        exit(true);
    end;

    protected procedure LoadData(ImportType: Enum "Import Type"): Boolean
    var
        Customer: Record Customer;
        ImportedCount: Integer;
    begin
        if TempCustomerBuffer.FindSet() then
            repeat
                Customer.TransferFields(TempCustomerBuffer);
                if Customer.Insert(true) then
                    ImportedCount += 1;
            until TempCustomerBuffer.Next() = 0;

        Message('%1 customers imported successfully', ImportedCount);
        exit(true);
    end;

    local procedure ParseCustomerLine(DataLine: List of [Text]): Boolean
    begin
        if DataLine.Count < 3 then
            exit(false);

        TempCustomerBuffer.Init();
        TempCustomerBuffer."No." := DataLine.Get(1);
        TempCustomerBuffer.Name := DataLine.Get(2);
        TempCustomerBuffer."Phone No." := DataLine.Get(3);
        if DataLine.Count > 3 then
            TempCustomerBuffer."E-Mail" := DataLine.Get(4);

        TempCustomerBuffer.Insert();
        exit(true);
    end;
}
```

## Workflow Template Pattern

```al
codeunit 50506 "Workflow Processing Template"
{
    procedure ProcessWorkflow(WorkflowCode: Code[20]; RecordVariant: Variant): Boolean
    var
        WorkflowStep: Record "Workflow Step";
        CurrentStepResult: Boolean;
    begin
        if not InitializeWorkflow(WorkflowCode, RecordVariant) then
            exit(false);

        WorkflowStep.SetRange("Workflow Code", WorkflowCode);
        WorkflowStep.SetRange(Active, true);
        WorkflowStep.SetCurrentKey("Step Order");

        if WorkflowStep.FindSet() then
            repeat
                CurrentStepResult := ExecuteWorkflowStep(WorkflowStep, RecordVariant);

                LogStepExecution(WorkflowStep, CurrentStepResult);

                if not CurrentStepResult then begin
                    HandleStepFailure(WorkflowStep, RecordVariant);
                    if WorkflowStep."Stop on Error" then
                        exit(false);
                end;

            until WorkflowStep.Next() = 0;

        exit(FinalizeWorkflow(WorkflowCode, RecordVariant));
    end;

    protected procedure InitializeWorkflow(WorkflowCode: Code[20]; RecordVariant: Variant): Boolean
    begin
        // Default initialization
        exit(true);
    end;

    protected procedure ExecuteWorkflowStep(WorkflowStep: Record "Workflow Step"; var RecordVariant: Variant): Boolean
    begin
        // Must be implemented by specific workflow types
        Error('ExecuteWorkflowStep must be implemented for step type %1', WorkflowStep."Step Type");
    end;

    protected procedure HandleStepFailure(WorkflowStep: Record "Workflow Step"; RecordVariant: Variant)
    begin
        // Default error handling
    end;

    protected procedure FinalizeWorkflow(WorkflowCode: Code[20]; RecordVariant: Variant): Boolean
    begin
        // Default finalization
        exit(true);
    end;
}
```

This sample demonstrates comprehensive template method pattern implementations including basic templates, document processing, configurable templates, data import patterns, and workflow processing with customizable steps and inheritance-based specialization.