# Facade Pattern AL Implementation - Code Examples

This sample demonstrates implementing facade patterns to simplify complex subsystems in Business Central.

## Basic Facade Interface

```al
// Simple customer management facade
codeunit 50400 "Customer Management Facade"
{
    procedure CreateCustomer(CustomerData: Dictionary of [Text, Variant]): Code[20]
    var
        Customer: Record Customer;
        CustomerNo: Code[20];
    begin
        CustomerNo := CreateNewCustomer(CustomerData);
        SetupCustomerDefaults(CustomerNo);
        InitializeCustomerRelationships(CustomerNo);

        exit(CustomerNo);
    end;

    procedure ProcessCustomerOrder(CustomerNo: Code[20]; OrderData: Dictionary of [Text, Variant]): Code[20]
    var
        SalesOrderNo: Code[20];
    begin
        ValidateCustomerEligibility(CustomerNo);
        SalesOrderNo := CreateSalesOrder(CustomerNo, OrderData);
        ApplyCustomerDiscounts(SalesOrderNo);
        SetupOrderApprovals(SalesOrderNo);

        exit(SalesOrderNo);
    end;

    procedure GetCustomerAnalytics(CustomerNo: Code[20]): Dictionary of [Text, Variant]
    var
        Analytics: Dictionary of [Text, Variant];
    begin
        Analytics := CalculateCustomerMetrics(CustomerNo);
        AddSalesHistory(CustomerNo, Analytics);
        AddPaymentHistory(CustomerNo, Analytics);
        AddCreditAnalysis(CustomerNo, Analytics);

        exit(Analytics);
    end;

    local procedure CreateNewCustomer(CustomerData: Dictionary of [Text, Variant]): Code[20]
    var
        Customer: Record Customer;
        CustomerName: Text[100];
        ContactName: Text[100];
    begin
        Customer.Init();
        Customer."No." := '';
        Customer.Insert(true);

        if CustomerData.Get('Name', CustomerName) then
            Customer.Validate(Name, CustomerName);

        if CustomerData.Get('Contact', ContactName) then
            Customer.Validate(Contact, ContactName);

        Customer.Modify(true);
        exit(Customer."No.");
    end;

    local procedure SetupCustomerDefaults(CustomerNo: Code[20])
    var
        Customer: Record Customer;
    begin
        Customer.Get(CustomerNo);
        Customer.Validate("Customer Posting Group", 'DOMESTIC');
        Customer.Validate("Gen. Bus. Posting Group", 'DOMESTIC');
        Customer.Validate("Payment Terms Code", '30 DAYS');
        Customer.Modify(true);
    end;

    local procedure InitializeCustomerRelationships(CustomerNo: Code[20])
    begin
        // Setup default contacts, ship-to addresses, etc.
        CreateDefaultShipToAddress(CustomerNo);
        AssignCustomerToSalesperson(CustomerNo);
    end;

    local procedure ValidateCustomerEligibility(CustomerNo: Code[20])
    var
        Customer: Record Customer;
    begin
        Customer.Get(CustomerNo);
        if Customer.Blocked <> Customer.Blocked::" " then
            Error('Customer %1 is blocked and cannot place orders', CustomerNo);

        CheckCreditLimit(Customer);
    end;

    local procedure CreateSalesOrder(CustomerNo: Code[20]; OrderData: Dictionary of [Text, Variant]): Code[20]
    var
        SalesHeader: Record "Sales Header";
        Items: List of [Dictionary of [Text, Variant]];
        ItemData: Dictionary of [Text, Variant];
    begin
        // Create sales header
        SalesHeader.Init();
        SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
        SalesHeader."No." := '';
        SalesHeader.Insert(true);
        SalesHeader.Validate("Sell-to Customer No.", CustomerNo);
        SalesHeader.Modify(true);

        // Add lines from order data
        if OrderData.Get('Items', Items) then begin
            foreach ItemData in Items do
                CreateSalesLine(SalesHeader, ItemData);
        end;

        exit(SalesHeader."No.");
    end;

    local procedure CreateSalesLine(var SalesHeader: Record "Sales Header"; ItemData: Dictionary of [Text, Variant])
    var
        SalesLine: Record "Sales Line";
        ItemNo: Code[20];
        Quantity: Decimal;
        NextLineNo: Integer;
    begin
        NextLineNo := GetNextLineNo(SalesHeader);

        SalesLine.Init();
        SalesLine."Document Type" := SalesHeader."Document Type";
        SalesLine."Document No." := SalesHeader."No.";
        SalesLine."Line No." := NextLineNo;
        SalesLine.Insert(true);

        if ItemData.Get('ItemNo', ItemNo) then
            SalesLine.Validate("No.", ItemNo);

        if ItemData.Get('Quantity', Quantity) then
            SalesLine.Validate(Quantity, Quantity);

        SalesLine.Modify(true);
    end;

    local procedure CalculateCustomerMetrics(CustomerNo: Code[20]): Dictionary of [Text, Variant]
    var
        Metrics: Dictionary of [Text, Variant];
        Customer: Record Customer;
        TotalSales: Decimal;
        OrderCount: Integer;
    begin
        Customer.Get(CustomerNo);
        Customer.CalcFields("Sales (LCY)", "Balance (LCY)");

        Metrics.Set('TotalSales', Customer."Sales (LCY)");
        Metrics.Set('CurrentBalance', Customer."Balance (LCY)");
        Metrics.Set('CreditLimit', Customer."Credit Limit (LCY)");
        Metrics.Set('LastOrderDate', GetLastOrderDate(CustomerNo));
        Metrics.Set('OrderCount', GetOrderCount(CustomerNo));

        exit(Metrics);
    end;

    // Additional helper procedures...
    local procedure CreateDefaultShipToAddress(CustomerNo: Code[20])
    begin
        // Implementation for default ship-to address
    end;

    local procedure AssignCustomerToSalesperson(CustomerNo: Code[20])
    begin
        // Implementation for salesperson assignment
    end;

    local procedure CheckCreditLimit(Customer: Record Customer)
    begin
        // Implementation for credit limit checking
    end;

    local procedure ApplyCustomerDiscounts(SalesOrderNo: Code[20])
    begin
        // Implementation for discount application
    end;

    local procedure SetupOrderApprovals(SalesOrderNo: Code[20])
    begin
        // Implementation for approval workflow setup
    end;

    local procedure AddSalesHistory(CustomerNo: Code[20]; var Analytics: Dictionary of [Text, Variant])
    begin
        // Implementation for sales history analysis
    end;

    local procedure AddPaymentHistory(CustomerNo: Code[20]; var Analytics: Dictionary of [Text, Variant])
    begin
        // Implementation for payment history analysis
    end;

    local procedure AddCreditAnalysis(CustomerNo: Code[20]; var Analytics: Dictionary of [Text, Variant])
    begin
        // Implementation for credit analysis
    end;

    local procedure GetNextLineNo(SalesHeader: Record "Sales Header"): Integer
    var
        SalesLine: Record "Sales Line";
    begin
        SalesLine.SetRange("Document Type", SalesHeader."Document Type");
        SalesLine.SetRange("Document No.", SalesHeader."No.");
        if SalesLine.FindLast() then
            exit(SalesLine."Line No." + 10000)
        else
            exit(10000);
    end;

    local procedure GetLastOrderDate(CustomerNo: Code[20]): Date
    var
        SalesHeader: Record "Sales Header";
    begin
        SalesHeader.SetRange("Sell-to Customer No.", CustomerNo);
        SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
        if SalesHeader.FindLast() then
            exit(SalesHeader."Order Date");

        exit(0D);
    end;

    local procedure GetOrderCount(CustomerNo: Code[20]): Integer
    var
        SalesHeader: Record "Sales Header";
    begin
        SalesHeader.SetRange("Sell-to Customer No.", CustomerNo);
        SalesHeader.SetRange("Document Type", SalesHeader."Document Type"::Order);
        exit(SalesHeader.Count);
    end;
}
```

## Integration Facade

```al
// Facade for external system integration
codeunit 50401 "External Integration Facade"
{
    var
        HttpClient: HttpClient;
        BaseUrl: Text;

    procedure SynchronizeCustomer(CustomerNo: Code[20]): Boolean
    var
        CustomerData: Dictionary of [Text, Variant];
        SyncResult: Boolean;
    begin
        CustomerData := ExtractCustomerData(CustomerNo);
        SyncResult := SendCustomerToExternalSystem(CustomerData);

        if SyncResult then
            UpdateSynchronizationStatus(CustomerNo, true)
        else
            LogSynchronizationError(CustomerNo);

        exit(SyncResult);
    end;

    procedure ImportExternalOrders(): Integer
    var
        ExternalOrders: List of [Dictionary of [Text, Variant]];
        OrderData: Dictionary of [Text, Variant];
        ImportedCount: Integer;
        CustomerFacade: Codeunit "Customer Management Facade";
    begin
        ExternalOrders := FetchOrdersFromExternalSystem();
        ImportedCount := 0;

        foreach OrderData in ExternalOrders do begin
            if ProcessExternalOrder(OrderData, CustomerFacade) then
                ImportedCount += 1;
        end;

        exit(ImportedCount);
    end;

    procedure ExportInventoryLevels(): Boolean
    var
        InventoryData: List of [Dictionary of [Text, Variant]];
        ExportResult: Boolean;
    begin
        InventoryData := ExtractInventoryData();
        ExportResult := SendInventoryToExternalSystem(InventoryData);

        if ExportResult then
            UpdateInventoryExportTimestamp();

        exit(ExportResult);
    end;

    local procedure ExtractCustomerData(CustomerNo: Code[20]): Dictionary of [Text, Variant]
    var
        Customer: Record Customer;
        CustomerData: Dictionary of [Text, Variant];
    begin
        Customer.Get(CustomerNo);

        CustomerData.Set('CustomerNumber', Customer."No.");
        CustomerData.Set('Name', Customer.Name);
        CustomerData.Set('Address', Customer.Address);
        CustomerData.Set('City', Customer.City);
        CustomerData.Set('PostCode', Customer."Post Code");
        CustomerData.Set('Country', Customer."Country/Region Code");
        CustomerData.Set('PhoneNumber', Customer."Phone No.");
        CustomerData.Set('Email', Customer."E-Mail");
        CustomerData.Set('CreditLimit', Customer."Credit Limit (LCY)");

        exit(CustomerData);
    end;

    local procedure SendCustomerToExternalSystem(CustomerData: Dictionary of [Text, Variant]): Boolean
    var
        JsonPayload: Text;
        HttpContent: HttpContent;
        HttpResponseMessage: HttpResponseMessage;
        Headers: HttpHeaders;
    begin
        JsonPayload := ConvertToJson(CustomerData);

        HttpContent.WriteFrom(JsonPayload);
        HttpContent.GetHeaders(Headers);
        Headers.Remove('Content-Type');
        Headers.Add('Content-Type', 'application/json');

        if HttpClient.Post(BaseUrl + '/api/customers', HttpContent, HttpResponseMessage) then
            exit(HttpResponseMessage.IsSuccessStatusCode)
        else
            exit(false);
    end;

    local procedure FetchOrdersFromExternalSystem(): List of [Dictionary of [Text, Variant]]
    var
        HttpResponseMessage: HttpResponseMessage;
        ResponseText: Text;
        Orders: List of [Dictionary of [Text, Variant]];
    begin
        if HttpClient.Get(BaseUrl + '/api/orders', HttpResponseMessage) then begin
            if HttpResponseMessage.IsSuccessStatusCode then begin
                HttpResponseMessage.Content.ReadAs(ResponseText);
                Orders := ParseOrdersFromJson(ResponseText);
            end;
        end;

        exit(Orders);
    end;

    local procedure ProcessExternalOrder(OrderData: Dictionary of [Text, Variant]; CustomerFacade: Codeunit "Customer Management Facade"): Boolean
    var
        CustomerNo: Code[20];
        OrderNo: Code[20];
    begin
        // Extract and validate customer
        if not OrderData.Get('CustomerNumber', CustomerNo) then
            exit(false);

        if not CustomerExists(CustomerNo) then
            CustomerNo := CreateCustomerFromOrderData(OrderData, CustomerFacade);

        // Process the order
        OrderNo := CustomerFacade.ProcessCustomerOrder(CustomerNo, OrderData);

        exit(OrderNo <> '');
    end;

    local procedure ExtractInventoryData(): List of [Dictionary of [Text, Variant]]
    var
        Item: Record Item;
        InventoryList: List of [Dictionary of [Text, Variant]];
        ItemData: Dictionary of [Text, Variant];
    begin
        Item.SetRange(Blocked, false);
        if Item.FindSet() then
            repeat
                Item.CalcFields(Inventory);

                Clear(ItemData);
                ItemData.Set('ItemNumber', Item."No.");
                ItemData.Set('Description', Item.Description);
                ItemData.Set('Inventory', Item.Inventory);
                ItemData.Set('UnitOfMeasure', Item."Base Unit of Measure");
                ItemData.Set('LastUpdated', CurrentDateTime);

                InventoryList.Add(ItemData);
            until Item.Next() = 0;

        exit(InventoryList);
    end;

    local procedure SendInventoryToExternalSystem(InventoryData: List of [Dictionary of [Text, Variant]]): Boolean
    var
        JsonPayload: Text;
        HttpContent: HttpContent;
        HttpResponseMessage: HttpResponseMessage;
        Headers: HttpHeaders;
    begin
        JsonPayload := ConvertInventoryToJson(InventoryData);

        HttpContent.WriteFrom(JsonPayload);
        HttpContent.GetHeaders(Headers);
        Headers.Remove('Content-Type');
        Headers.Add('Content-Type', 'application/json');

        if HttpClient.Post(BaseUrl + '/api/inventory', HttpContent, HttpResponseMessage) then
            exit(HttpResponseMessage.IsSuccessStatusCode)
        else
            exit(false);
    end;

    // Helper procedures
    local procedure ConvertToJson(Data: Dictionary of [Text, Variant]): Text
    begin
        // JSON conversion implementation
        exit('{}'); // Placeholder
    end;

    local procedure ParseOrdersFromJson(JsonText: Text): List of [Dictionary of [Text, Variant]]
    var
        Orders: List of [Dictionary of [Text, Variant]];
    begin
        // JSON parsing implementation
        exit(Orders);
    end;

    local procedure ConvertInventoryToJson(InventoryData: List of [Dictionary of [Text, Variant]]): Text
    begin
        // JSON conversion for inventory data
        exit('[]'); // Placeholder
    end;

    local procedure CustomerExists(CustomerNo: Code[20]): Boolean
    var
        Customer: Record Customer;
    begin
        exit(Customer.Get(CustomerNo));
    end;

    local procedure CreateCustomerFromOrderData(OrderData: Dictionary of [Text, Variant]; CustomerFacade: Codeunit "Customer Management Facade"): Code[20]
    var
        CustomerData: Dictionary of [Text, Variant];
    begin
        // Extract customer info from order data
        CustomerData.Set('Name', OrderData.Get('CustomerName'));
        CustomerData.Set('Contact', OrderData.Get('ContactName'));

        exit(CustomerFacade.CreateCustomer(CustomerData));
    end;

    local procedure UpdateSynchronizationStatus(CustomerNo: Code[20]; Success: Boolean)
    begin
        // Update sync status tracking
    end;

    local procedure LogSynchronizationError(CustomerNo: Code[20])
    begin
        // Log sync errors for troubleshooting
    end;

    local procedure UpdateInventoryExportTimestamp()
    begin
        // Update last export timestamp
    end;
}
```

## Multi-Layer Business Process Facade

```al
// Complex facade coordinating multiple business processes
codeunit 50402 "Sales Process Facade"
{
    var
        CustomerFacade: Codeunit "Customer Management Facade";
        IntegrationFacade: Codeunit "External Integration Facade";

    procedure ProcessCompleteOrder(OrderRequest: Dictionary of [Text, Variant]): Dictionary of [Text, Variant]
    var
        ProcessingResult: Dictionary of [Text, Variant];
        CustomerNo: Code[20];
        OrderNo: Code[20];
        ProcessingSteps: List of [Text];
    begin
        ProcessingSteps.Add('Validate Customer');
        ProcessingSteps.Add('Create/Update Customer');
        ProcessingSteps.Add('Process Order');
        ProcessingSteps.Add('Apply Business Rules');
        ProcessingSteps.Add('External Synchronization');
        ProcessingSteps.Add('Generate Confirmations');

        ProcessingResult.Set('Steps', ProcessingSteps);
        ProcessingResult.Set('StartTime', CurrentDateTime);

        try
            // Step 1: Customer handling
            CustomerNo := HandleCustomerProcess(OrderRequest, ProcessingResult);

            // Step 2: Order processing
            OrderNo := HandleOrderProcess(CustomerNo, OrderRequest, ProcessingResult);

            // Step 3: Business rules and approvals
            HandleBusinessRulesProcess(OrderNo, ProcessingResult);

            // Step 4: External integration
            HandleExternalIntegration(CustomerNo, OrderNo, ProcessingResult);

            // Step 5: Confirmations and notifications
            HandleConfirmationProcess(OrderNo, ProcessingResult);

            ProcessingResult.Set('Success', true);
            ProcessingResult.Set('OrderNumber', OrderNo);
            ProcessingResult.Set('CustomerNumber', CustomerNo);

        except
            ProcessingResult.Set('Success', false);
            ProcessingResult.Set('Error', GetLastErrorText);
        end;

        ProcessingResult.Set('EndTime', CurrentDateTime);
        exit(ProcessingResult);
    end;

    procedure GetOrderStatus(OrderNo: Code[20]): Dictionary of [Text, Variant]
    var
        StatusInfo: Dictionary of [Text, Variant];
        SalesHeader: Record "Sales Header";
    begin
        if SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo) then begin
            StatusInfo.Set('OrderNumber', OrderNo);
            StatusInfo.Set('Status', Format(SalesHeader.Status));
            StatusInfo.Set('CustomerNumber', SalesHeader."Sell-to Customer No.");
            StatusInfo.Set('OrderDate', SalesHeader."Order Date");
            StatusInfo.Set('Amount', SalesHeader."Amount Including VAT");
            StatusInfo.Set('PaymentTerms', SalesHeader."Payment Terms Code");

            // Add processing history
            AddProcessingHistory(OrderNo, StatusInfo);
        end else begin
            StatusInfo.Set('Found', false);
            StatusInfo.Set('Error', 'Order not found');
        end;

        exit(StatusInfo);
    end;

    local procedure HandleCustomerProcess(OrderRequest: Dictionary of [Text, Variant]; var ProcessingResult: Dictionary of [Text, Variant]): Code[20]
    var
        CustomerNo: Code[20];
        CustomerData: Dictionary of [Text, Variant];
    begin
        ProcessingResult.Set('CurrentStep', 'Processing Customer');

        if OrderRequest.Get('CustomerNumber', CustomerNo) then begin
            if not CustomerExists(CustomerNo) then begin
                ExtractCustomerDataFromOrder(OrderRequest, CustomerData);
                CustomerNo := CustomerFacade.CreateCustomer(CustomerData);
                ProcessingResult.Set('CustomerCreated', true);
            end else begin
                ProcessingResult.Set('CustomerCreated', false);
            end;
        end else begin
            Error('Customer number is required for order processing');
        end;

        ProcessingResult.Set('CustomerNumber', CustomerNo);
        exit(CustomerNo);
    end;

    local procedure HandleOrderProcess(CustomerNo: Code[20]; OrderRequest: Dictionary of [Text, Variant]; var ProcessingResult: Dictionary of [Text, Variant]): Code[20]
    var
        OrderNo: Code[20];
    begin
        ProcessingResult.Set('CurrentStep', 'Processing Order');

        OrderNo := CustomerFacade.ProcessCustomerOrder(CustomerNo, OrderRequest);

        if OrderNo <> '' then begin
            ProcessingResult.Set('OrderCreated', true);
            ProcessingResult.Set('OrderNumber', OrderNo);
        end else begin
            Error('Failed to create sales order');
        end;

        exit(OrderNo);
    end;

    local procedure HandleBusinessRulesProcess(OrderNo: Code[20]; var ProcessingResult: Dictionary of [Text, Variant])
    var
        ApprovalRequired: Boolean;
        CreditCheckPassed: Boolean;
    begin
        ProcessingResult.Set('CurrentStep', 'Applying Business Rules');

        // Check if approval is required
        ApprovalRequired := CheckApprovalRequirement(OrderNo);
        ProcessingResult.Set('ApprovalRequired', ApprovalRequired);

        // Perform credit check
        CreditCheckPassed := PerformCreditCheck(OrderNo);
        ProcessingResult.Set('CreditCheckPassed', CreditCheckPassed);

        if ApprovalRequired then
            InitiateApprovalWorkflow(OrderNo);

        if not CreditCheckPassed then
            Error('Credit check failed for order %1', OrderNo);
    end;

    local procedure HandleExternalIntegration(CustomerNo: Code[20]; OrderNo: Code[20]; var ProcessingResult: Dictionary of [Text, Variant])
    var
        SyncResult: Boolean;
    begin
        ProcessingResult.Set('CurrentStep', 'External Synchronization');

        // Synchronize customer with external system
        SyncResult := IntegrationFacade.SynchronizeCustomer(CustomerNo);
        ProcessingResult.Set('CustomerSyncSuccess', SyncResult);

        // Additional integration as needed
        ProcessingResult.Set('ExternalIntegrationCompleted', true);
    end;

    local procedure HandleConfirmationProcess(OrderNo: Code[20]; var ProcessingResult: Dictionary of [Text, Variant])
    begin
        ProcessingResult.Set('CurrentStep', 'Generating Confirmations');

        SendOrderConfirmation(OrderNo);
        NotifyWarehouse(OrderNo);
        UpdateCustomerStatistics(OrderNo);

        ProcessingResult.Set('ConfirmationsGenerated', true);
    end;

    // Additional helper procedures...
    local procedure CustomerExists(CustomerNo: Code[20]): Boolean
    var
        Customer: Record Customer;
    begin
        exit(Customer.Get(CustomerNo));
    end;

    local procedure ExtractCustomerDataFromOrder(OrderRequest: Dictionary of [Text, Variant]; var CustomerData: Dictionary of [Text, Variant])
    begin
        // Extract customer information from order request
        CustomerData.Set('Name', OrderRequest.Get('CustomerName'));
        CustomerData.Set('Contact', OrderRequest.Get('ContactName'));
        CustomerData.Set('Address', OrderRequest.Get('CustomerAddress'));
    end;

    local procedure CheckApprovalRequirement(OrderNo: Code[20]): Boolean
    var
        SalesHeader: Record "Sales Header";
    begin
        if SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo) then
            exit(SalesHeader."Amount Including VAT" > 10000); // Approval required for orders > 10,000

        exit(false);
    end;

    local procedure PerformCreditCheck(OrderNo: Code[20]): Boolean
    begin
        // Credit check implementation
        exit(true); // Simplified for example
    end;

    local procedure InitiateApprovalWorkflow(OrderNo: Code[20])
    begin
        // Approval workflow initiation
    end;

    local procedure SendOrderConfirmation(OrderNo: Code[20])
    begin
        // Order confirmation implementation
    end;

    local procedure NotifyWarehouse(OrderNo: Code[20])
    begin
        // Warehouse notification implementation
    end;

    local procedure UpdateCustomerStatistics(OrderNo: Code[20])
    begin
        // Customer statistics update implementation
    end;

    local procedure AddProcessingHistory(OrderNo: Code[20]; var StatusInfo: Dictionary of [Text, Variant])
    begin
        // Add processing history to status information
    end;
}
```

This comprehensive facade implementation demonstrates how to simplify complex business processes by providing clean, unified interfaces that coordinate multiple subsystems while hiding implementation complexity from consumers.