# API Interface Design Patterns - AL Code Examples

This sample demonstrates effective API interface design patterns for maintainable, extensible APIs in Business Central.

## Basic Interface Design Patterns

```al
// Clean, focused interface for customer operations
interface ICustomerService
{
    procedure CreateCustomer(CustomerData: Dictionary of [Text, Variant]): Code[20];
    procedure UpdateCustomer(CustomerNo: Code[20]; UpdateData: Dictionary of [Text, Variant]): Boolean;
    procedure GetCustomer(CustomerNo: Code[20]): Dictionary of [Text, Variant];
    procedure DeleteCustomer(CustomerNo: Code[20]): Boolean;
}

// Separate interface for customer queries (Command-Query Separation)
interface ICustomerQuery
{
    procedure FindCustomersByName(NamePattern: Text): List of [Code[20]];
    procedure GetCustomerSalesHistory(CustomerNo: Code[20]; FromDate: Date; ToDate: Date): List of [Dictionary of [Text, Variant]];
    procedure GetCustomerBalance(CustomerNo: Code[20]): Decimal;
    procedure ValidateCustomerExists(CustomerNo: Code[20]): Boolean;
}
```

## Fluent Interface Pattern

```al
// Fluent interface for sales order creation
interface ISalesOrderBuilder
{
    procedure ForCustomer(CustomerNo: Code[20]): Interface ISalesOrderBuilder;
    procedure WithOrderDate(OrderDate: Date): Interface ISalesOrderBuilder;
    procedure AddItem(ItemNo: Code[20]; Quantity: Decimal): Interface ISalesOrderBuilder;
    procedure AddItemWithPrice(ItemNo: Code[20]; Quantity: Decimal; UnitPrice: Decimal): Interface ISalesOrderBuilder;
    procedure WithShipToAddress(Address: Dictionary of [Text, Variant]): Interface ISalesOrderBuilder;
    procedure WithPaymentTerms(PaymentTermsCode: Code[10]): Interface ISalesOrderBuilder;
    procedure Build(): Code[20];
}

// Implementation of fluent interface
codeunit 50600 "Sales Order Builder" implements ISalesOrderBuilder
{
    var
        CustomerNo: Code[20];
        OrderDate: Date;
        Items: List of [Dictionary of [Text, Variant]];
        ShipToAddress: Dictionary of [Text, Variant];
        PaymentTermsCode: Code[10];

    procedure ForCustomer(NewCustomerNo: Code[20]): Interface ISalesOrderBuilder
    begin
        CustomerNo := NewCustomerNo;
        exit(this);
    end;

    procedure WithOrderDate(NewOrderDate: Date): Interface ISalesOrderBuilder
    begin
        OrderDate := NewOrderDate;
        exit(this);
    end;

    procedure AddItem(ItemNo: Code[20]; Quantity: Decimal): Interface ISalesOrderBuilder
    var
        ItemData: Dictionary of [Text, Variant];
    begin
        ItemData.Set('ItemNo', ItemNo);
        ItemData.Set('Quantity', Quantity);
        Items.Add(ItemData);
        exit(this);
    end;

    procedure AddItemWithPrice(ItemNo: Code[20]; Quantity: Decimal; UnitPrice: Decimal): Interface ISalesOrderBuilder
    var
        ItemData: Dictionary of [Text, Variant];
    begin
        ItemData.Set('ItemNo', ItemNo);
        ItemData.Set('Quantity', Quantity);
        ItemData.Set('UnitPrice', UnitPrice);
        Items.Add(ItemData);
        exit(this);
    end;

    procedure WithShipToAddress(Address: Dictionary of [Text, Variant]): Interface ISalesOrderBuilder
    begin
        ShipToAddress := Address;
        exit(this);
    end;

    procedure WithPaymentTerms(NewPaymentTermsCode: Code[10]): Interface ISalesOrderBuilder
    begin
        PaymentTermsCode := NewPaymentTermsCode;
        exit(this);
    end;

    procedure Build(): Code[20]
    var
        SalesHeader: Record "Sales Header";
        SalesLine: Record "Sales Line";
        ItemData: Dictionary of [Text, Variant];
        LineNo: Integer;
    begin
        if CustomerNo = '' then
            Error('Customer number is required');

        // Create sales header
        SalesHeader.Init();
        SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
        SalesHeader."No." := '';
        SalesHeader.Insert(true);

        SalesHeader.Validate("Sell-to Customer No.", CustomerNo);

        if OrderDate <> 0D then
            SalesHeader.Validate("Order Date", OrderDate);

        if PaymentTermsCode <> '' then
            SalesHeader.Validate("Payment Terms Code", PaymentTermsCode);

        SalesHeader.Modify(true);

        // Add ship-to address if specified
        if ShipToAddress.Count > 0 then
            ApplyShipToAddress(SalesHeader, ShipToAddress);

        // Add items
        LineNo := 10000;
        foreach ItemData in Items do begin
            CreateSalesLine(SalesHeader, ItemData, LineNo);
            LineNo += 10000;
        end;

        exit(SalesHeader."No.");
    end;

    local procedure ApplyShipToAddress(var SalesHeader: Record "Sales Header"; Address: Dictionary of [Text, Variant])
    var
        ShipToName: Text[100];
        ShipToAddress1: Text[100];
        ShipToCity: Text[30];
    begin
        if Address.Get('Name', ShipToName) then
            SalesHeader.Validate("Ship-to Name", ShipToName);

        if Address.Get('Address', ShipToAddress1) then
            SalesHeader.Validate("Ship-to Address", ShipToAddress1);

        if Address.Get('City', ShipToCity) then
            SalesHeader.Validate("Ship-to City", ShipToCity);

        SalesHeader.Modify(true);
    end;

    local procedure CreateSalesLine(SalesHeader: Record "Sales Header"; ItemData: Dictionary of [Text, Variant]; LineNo: Integer)
    var
        SalesLine: Record "Sales Line";
        ItemNo: Code[20];
        Quantity: Decimal;
        UnitPrice: Decimal;
    begin
        SalesLine.Init();
        SalesLine."Document Type" := SalesHeader."Document Type";
        SalesLine."Document No." := SalesHeader."No.";
        SalesLine."Line No." := LineNo;
        SalesLine.Insert(true);

        if ItemData.Get('ItemNo', ItemNo) then
            SalesLine.Validate("No.", ItemNo);

        if ItemData.Get('Quantity', Quantity) then
            SalesLine.Validate(Quantity, Quantity);

        if ItemData.Get('UnitPrice', UnitPrice) then
            SalesLine.Validate("Unit Price", UnitPrice);

        SalesLine.Modify(true);
    end;
}
```

## Result Pattern for Error Handling

```al
// Result pattern for robust error handling
interface IOperationResult
{
    procedure IsSuccess(): Boolean;
    procedure IsFailure(): Boolean;
    procedure GetErrorMessage(): Text;
    procedure GetErrorCode(): Text[20];
    procedure GetData(): Dictionary of [Text, Variant];
}

// Implementation of operation result
codeunit 50601 "Operation Result" implements IOperationResult
{
    var
        Success: Boolean;
        ErrorMessage: Text;
        ErrorCode: Text[20];
        Data: Dictionary of [Text, Variant];

    procedure Initialize(IsSuccessful: Boolean; Message: Text; Code: Text[20]; ResultData: Dictionary of [Text, Variant])
    begin
        Success := IsSuccessful;
        ErrorMessage := Message;
        ErrorCode := Code;
        Data := ResultData;
    end;

    procedure IsSuccess(): Boolean
    begin
        exit(Success);
    end;

    procedure IsFailure(): Boolean
    begin
        exit(not Success);
    end;

    procedure GetErrorMessage(): Text
    begin
        exit(ErrorMessage);
    end;

    procedure GetErrorCode(): Text[20]
    begin
        exit(ErrorCode);
    end;

    procedure GetData(): Dictionary of [Text, Variant]
    begin
        exit(Data);
    end;

    procedure CreateSuccess(ResultData: Dictionary of [Text, Variant]): Interface IOperationResult
    var
        Result: Codeunit "Operation Result";
    begin
        Result.Initialize(true, '', '', ResultData);
        exit(Result);
    end;

    procedure CreateFailure(Message: Text; Code: Text[20]): Interface IOperationResult
    var
        Result: Codeunit "Operation Result";
        EmptyData: Dictionary of [Text, Variant];
    begin
        Result.Initialize(false, Message, Code, EmptyData);
        exit(Result);
    end;
}
```

## Service Interface with Result Pattern

```al
// Enhanced customer service using result pattern
interface IAdvancedCustomerService
{
    procedure CreateCustomer(CustomerData: Dictionary of [Text, Variant]): Interface IOperationResult;
    procedure UpdateCustomer(CustomerNo: Code[20]; UpdateData: Dictionary of [Text, Variant]): Interface IOperationResult;
    procedure GetCustomer(CustomerNo: Code[20]): Interface IOperationResult;
    procedure ValidateCustomerData(CustomerData: Dictionary of [Text, Variant]): Interface IOperationResult;
}

// Implementation with comprehensive error handling
codeunit 50602 "Advanced Customer Service" implements IAdvancedCustomerService
{
    procedure CreateCustomer(CustomerData: Dictionary of [Text, Variant]): Interface IOperationResult
    var
        Customer: Record Customer;
        ValidationResult: Interface IOperationResult;
        CustomerNo: Code[20];
        ResultData: Dictionary of [Text, Variant];
        OperationResult: Codeunit "Operation Result";
    begin
        // Validate input data
        ValidationResult := ValidateCustomerData(CustomerData);
        if ValidationResult.IsFailure() then
            exit(ValidationResult);

        try
            // Create customer
            Customer.Init();
            Customer."No." := '';
            Customer.Insert(true);
            CustomerNo := Customer."No.";

            // Apply data
            ApplyCustomerData(Customer, CustomerData);

            // Prepare success result
            ResultData.Set('CustomerNo', CustomerNo);
            ResultData.Set('Created', true);

            exit(OperationResult.CreateSuccess(ResultData));

        except
            exit(OperationResult.CreateFailure(GetLastErrorText(), 'CREATE_FAILED'));
        end;
    end;

    procedure UpdateCustomer(CustomerNo: Code[20]; UpdateData: Dictionary of [Text, Variant]): Interface IOperationResult
    var
        Customer: Record Customer;
        ValidationResult: Interface IOperationResult;
        ResultData: Dictionary of [Text, Variant];
        OperationResult: Codeunit "Operation Result";
    begin
        if not Customer.Get(CustomerNo) then
            exit(OperationResult.CreateFailure(StrSubstNo('Customer %1 not found', CustomerNo), 'NOT_FOUND'));

        // Validate update data
        ValidationResult := ValidateCustomerData(UpdateData);
        if ValidationResult.IsFailure() then
            exit(ValidationResult);

        try
            ApplyCustomerData(Customer, UpdateData);

            ResultData.Set('CustomerNo', CustomerNo);
            ResultData.Set('Updated', true);

            exit(OperationResult.CreateSuccess(ResultData));

        except
            exit(OperationResult.CreateFailure(GetLastErrorText(), 'UPDATE_FAILED'));
        end;
    end;

    procedure GetCustomer(CustomerNo: Code[20]): Interface IOperationResult
    var
        Customer: Record Customer;
        CustomerData: Dictionary of [Text, Variant];
        OperationResult: Codeunit "Operation Result";
    begin
        if not Customer.Get(CustomerNo) then
            exit(OperationResult.CreateFailure(StrSubstNo('Customer %1 not found', CustomerNo), 'NOT_FOUND'));

        CustomerData := ExtractCustomerData(Customer);
        exit(OperationResult.CreateSuccess(CustomerData));
    end;

    procedure ValidateCustomerData(CustomerData: Dictionary of [Text, Variant]): Interface IOperationResult
    var
        CustomerName: Text[100];
        ValidationErrors: List of [Text];
        ErrorMessage: Text;
        OperationResult: Codeunit "Operation Result";
        ResultData: Dictionary of [Text, Variant];
    begin
        // Validate required fields
        if not CustomerData.Get('Name', CustomerName) or (CustomerName = '') then
            ValidationErrors.Add('Customer name is required');

        if CustomerName <> '' then begin
            if StrLen(CustomerName) > 100 then
                ValidationErrors.Add('Customer name cannot exceed 100 characters');
        end;

        // Additional validation logic...

        if ValidationErrors.Count > 0 then begin
            ErrorMessage := JoinErrors(ValidationErrors);
            exit(OperationResult.CreateFailure(ErrorMessage, 'VALIDATION_FAILED'));
        end;

        ResultData.Set('Valid', true);
        exit(OperationResult.CreateSuccess(ResultData));
    end;

    local procedure ApplyCustomerData(var Customer: Record Customer; CustomerData: Dictionary of [Text, Variant])
    var
        CustomerName: Text[100];
        Address: Text[100];
        City: Text[30];
        PhoneNo: Text[30];
        Email: Text[80];
    begin
        if CustomerData.Get('Name', CustomerName) then
            Customer.Validate(Name, CustomerName);

        if CustomerData.Get('Address', Address) then
            Customer.Validate(Address, Address);

        if CustomerData.Get('City', City) then
            Customer.Validate(City, City);

        if CustomerData.Get('PhoneNo', PhoneNo) then
            Customer.Validate("Phone No.", PhoneNo);

        if CustomerData.Get('Email', Email) then
            Customer.Validate("E-Mail", Email);

        Customer.Modify(true);
    end;

    local procedure ExtractCustomerData(Customer: Record Customer): Dictionary of [Text, Variant]
    var
        CustomerData: Dictionary of [Text, Variant];
    begin
        CustomerData.Set('CustomerNo', Customer."No.");
        CustomerData.Set('Name', Customer.Name);
        CustomerData.Set('Address', Customer.Address);
        CustomerData.Set('City', Customer.City);
        CustomerData.Set('PostCode', Customer."Post Code");
        CustomerData.Set('PhoneNo', Customer."Phone No.");
        CustomerData.Set('Email', Customer."E-Mail");
        CustomerData.Set('CreditLimit', Customer."Credit Limit (LCY)");
        CustomerData.Set('Blocked', Customer.Blocked);

        exit(CustomerData);
    end;

    local procedure JoinErrors(Errors: List of [Text]): Text
    var
        ErrorMessage: Text;
        Error: Text;
    begin
        foreach Error in Errors do begin
            if ErrorMessage <> '' then
                ErrorMessage += '; ';
            ErrorMessage += Error;
        end;

        exit(ErrorMessage);
    end;
}
```

## Versioned Interface Pattern

```al
// Interface versioning for backward compatibility
interface ICustomerServiceV1
{
    procedure CreateCustomer(Name: Text[100]; Address: Text[100]): Code[20];
    procedure GetCustomerName(CustomerNo: Code[20]): Text[100];
}

interface ICustomerServiceV2 extends ICustomerServiceV1
{
    procedure CreateCustomerAdvanced(CustomerData: Dictionary of [Text, Variant]): Interface IOperationResult;
    procedure GetCustomerDetails(CustomerNo: Code[20]): Dictionary of [Text, Variant];
    procedure UpdateCustomer(CustomerNo: Code[20]; UpdateData: Dictionary of [Text, Variant]): Interface IOperationResult;
}

// Implementation supporting both versions
codeunit 50603 "Customer Service V2" implements ICustomerServiceV2
{
    // V1 methods for backward compatibility
    procedure CreateCustomer(Name: Text[100]; Address: Text[100]): Code[20]
    var
        CustomerData: Dictionary of [Text, Variant];
        Result: Interface IOperationResult;
        CustomerNo: Code[20];
    begin
        CustomerData.Set('Name', Name);
        CustomerData.Set('Address', Address);

        Result := CreateCustomerAdvanced(CustomerData);
        if Result.IsSuccess() then begin
            Result.GetData().Get('CustomerNo', CustomerNo);
            exit(CustomerNo);
        end else begin
            Error(Result.GetErrorMessage());
        end;
    end;

    procedure GetCustomerName(CustomerNo: Code[20]): Text[100]
    var
        CustomerDetails: Dictionary of [Text, Variant];
        CustomerName: Text[100];
    begin
        CustomerDetails := GetCustomerDetails(CustomerNo);
        if CustomerDetails.Get('Name', CustomerName) then
            exit(CustomerName);

        exit('');
    end;

    // V2 methods with enhanced functionality
    procedure CreateCustomerAdvanced(CustomerData: Dictionary of [Text, Variant]): Interface IOperationResult
    var
        AdvancedService: Codeunit "Advanced Customer Service";
    begin
        exit(AdvancedService.CreateCustomer(CustomerData));
    end;

    procedure GetCustomerDetails(CustomerNo: Code[20]): Dictionary of [Text, Variant]
    var
        AdvancedService: Codeunit "Advanced Customer Service";
        Result: Interface IOperationResult;
        EmptyData: Dictionary of [Text, Variant];
    begin
        Result := AdvancedService.GetCustomer(CustomerNo);
        if Result.IsSuccess() then
            exit(Result.GetData())
        else
            exit(EmptyData);
    end;

    procedure UpdateCustomer(CustomerNo: Code[20]; UpdateData: Dictionary of [Text, Variant]): Interface IOperationResult
    var
        AdvancedService: Codeunit "Advanced Customer Service";
    begin
        exit(AdvancedService.UpdateCustomer(CustomerNo, UpdateData));
    end;
}
```

## Usage Example

```al
// Example of using the designed interfaces
codeunit 50604 "API Usage Example"
{
    procedure DemonstrateAPIUsage()
    var
        SalesOrderBuilder: Codeunit "Sales Order Builder";
        CustomerService: Codeunit "Advanced Customer Service";
        CustomerData: Dictionary of [Text, Variant];
        ShipToAddress: Dictionary of [Text, Variant];
        Result: Interface IOperationResult;
        OrderNo: Code[20];
        CustomerNo: Code[20];
    begin
        // Create customer using advanced service
        CustomerData.Set('Name', 'Example Customer');
        CustomerData.Set('Address', '123 Main Street');
        CustomerData.Set('City', 'Example City');
        CustomerData.Set('PhoneNo', '555-0123');
        CustomerData.Set('Email', 'customer@example.com');

        Result := CustomerService.CreateCustomer(CustomerData);

        if Result.IsFailure() then begin
            Message('Customer creation failed: %1', Result.GetErrorMessage());
            exit;
        end;

        Result.GetData().Get('CustomerNo', CustomerNo);

        // Create sales order using fluent interface
        ShipToAddress.Set('Name', 'Shipping Department');
        ShipToAddress.Set('Address', '456 Shipping Lane');
        ShipToAddress.Set('City', 'Warehouse City');

        OrderNo := SalesOrderBuilder
            .ForCustomer(CustomerNo)
            .WithOrderDate(Today)
            .AddItem('ITEM001', 5)
            .AddItemWithPrice('ITEM002', 10, 25.50)
            .WithShipToAddress(ShipToAddress)
            .WithPaymentTerms('30 DAYS')
            .Build();

        Message('Sales order %1 created successfully for customer %2', OrderNo, CustomerNo);
    end;
}
```

This comprehensive example demonstrates clean API interface design with separation of concerns, fluent interfaces, robust error handling using the result pattern, and interface versioning for backward compatibility.