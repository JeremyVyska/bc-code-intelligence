# API Delegate Operation Pattern - AL Code Examples

This sample demonstrates implementing delegate operations for extensible APIs in Business Central.

## Basic Delegate Interface

```al
// Define delegate operation interface
interface IDelegateOperation
{
    procedure Execute(var Parameters: Dictionary of [Text, Variant]): Boolean;
    procedure GetName(): Text[50];
    procedure GetDescription(): Text[250];
}
```

## Simple Delegate Implementation

```al
// Customer validation delegate example
codeunit 50100 "Customer Validation Delegate" implements IDelegateOperation
{
    procedure Execute(var Parameters: Dictionary of [Text, Variant]): Boolean
    var
        Customer: Record Customer;
        CustomerNo: Code[20];
        ValidationResult: Boolean;
    begin
        // Extract parameters
        if not Parameters.Get('CustomerNo', CustomerNo) then
            exit(false);

        if not Customer.Get(CustomerNo) then
            exit(false);

        // Custom validation logic
        ValidationResult := ValidateCustomerForOperation(Customer);
        Parameters.Set('IsValid', ValidationResult);

        exit(ValidationResult);
    end;

    procedure GetName(): Text[50]
    begin
        exit('CUSTOMER_VALIDATION');
    end;

    procedure GetDescription(): Text[250]
    begin
        exit('Validates customer eligibility for business operations');
    end;

    local procedure ValidateCustomerForOperation(Customer: Record Customer): Boolean
    begin
        // Business logic for customer validation
        exit((Customer.Blocked = Customer.Blocked::" ") and (Customer."Credit Limit (LCY)" > 0));
    end;
}
```

## Delegate Registry System

```al
// Central delegate registry for operation management
codeunit 50101 "Delegate Registry"
{
    var
        DelegateOperations: Dictionary of [Text, Interface IDelegateOperation];

    procedure RegisterDelegate(OperationType: Text[50]; DelegateOperation: Interface IDelegateOperation)
    begin
        DelegateOperations.Set(OperationType, DelegateOperation);
    end;

    procedure ExecuteDelegate(OperationType: Text[50]; var Parameters: Dictionary of [Text, Variant]): Boolean
    var
        DelegateOperation: Interface IDelegateOperation;
    begin
        if not DelegateOperations.Get(OperationType, DelegateOperation) then
            exit(false);

        exit(DelegateOperation.Execute(Parameters));
    end;

    procedure GetAvailableDelegates(): List of [Text]
    var
        AvailableTypes: List of [Text];
        OperationType: Text;
    begin
        foreach OperationType in DelegateOperations.Keys do
            AvailableTypes.Add(OperationType);

        exit(AvailableTypes);
    end;
}
```

## API with Delegate Operations

```al
// Sales API that uses delegate operations for extensibility
page 50100 "Sales API"
{
    PageType = API;
    APIPublisher = 'mycompany';
    APIGroup = 'sales';
    APIVersion = 'v1.0';
    EntityName = 'salesOrder';
    EntitySetName = 'salesOrders';
    SourceTable = "Sales Header";

    layout
    {
        area(Content)
        {
            repeater(Group)
            {
                field(documentNo; Rec."No.")
                {
                    ApplicationArea = All;
                }
                field(customerNo; Rec."Sell-to Customer No.")
                {
                    ApplicationArea = All;
                }
                field(status; Rec.Status)
                {
                    ApplicationArea = All;
                }
            }
        }
    }

    trigger OnInsertRecord(BelowxRec: Boolean): Boolean
    var
        DelegateRegistry: Codeunit "Delegate Registry";
        Parameters: Dictionary of [Text, Variant];
        ValidationResult: Boolean;
    begin
        // Use delegate for customer validation
        Parameters.Set('CustomerNo', Rec."Sell-to Customer No.");
        ValidationResult := DelegateRegistry.ExecuteDelegate('CUSTOMER_VALIDATION', Parameters);

        if not ValidationResult then
            Error('Customer validation failed through delegate operation');

        exit(true);
    end;
}
```

## Advanced Delegate Chain

```al
// Chain multiple delegates for complex operations
codeunit 50102 "Multi-Step Delegate Chain"
{
    var
        DelegateRegistry: Codeunit "Delegate Registry";

    procedure ExecuteChain(ChainName: Text[50]; var Parameters: Dictionary of [Text, Variant]): Boolean
    var
        ChainSteps: List of [Text];
        StepName: Text;
        StepResult: Boolean;
    begin
        ChainSteps := GetChainSteps(ChainName);

        foreach StepName in ChainSteps do begin
            StepResult := DelegateRegistry.ExecuteDelegate(StepName, Parameters);
            if not StepResult then begin
                Parameters.Set('FailedStep', StepName);
                exit(false);
            end;
        end;

        exit(true);
    end;

    local procedure GetChainSteps(ChainName: Text[50]): List of [Text]
    var
        Steps: List of [Text];
    begin
        case ChainName of
            'SALES_VALIDATION':
                begin
                    Steps.Add('CUSTOMER_VALIDATION');
                    Steps.Add('INVENTORY_CHECK');
                    Steps.Add('CREDIT_APPROVAL');
                end;
            'PURCHASE_VALIDATION':
                begin
                    Steps.Add('VENDOR_VALIDATION');
                    Steps.Add('BUDGET_CHECK');
                    Steps.Add('APPROVAL_WORKFLOW');
                end;
        end;

        exit(Steps);
    end;
}
```

## Error Handling Delegate

```al
// Delegate with comprehensive error handling
codeunit 50103 "Error Handling Delegate" implements IDelegateOperation
{
    procedure Execute(var Parameters: Dictionary of [Text, Variant]): Boolean
    var
        ErrorMessages: List of [Text];
        HasErrors: Boolean;
    begin
        HasErrors := false;
        Clear(ErrorMessages);

        // Perform validation with error collection
        if not ValidateRequiredParameters(Parameters, ErrorMessages) then
            HasErrors := true;

        if not ValidateBusinessRules(Parameters, ErrorMessages) then
            HasErrors := true;

        // Set error information in parameters
        Parameters.Set('HasErrors', HasErrors);
        Parameters.Set('ErrorMessages', ErrorMessages);

        exit(not HasErrors);
    end;

    procedure GetName(): Text[50]
    begin
        exit('ERROR_HANDLING');
    end;

    procedure GetDescription(): Text[250]
    begin
        exit('Comprehensive error handling and validation delegate');
    end;

    local procedure ValidateRequiredParameters(Parameters: Dictionary of [Text, Variant]; var ErrorMessages: List of [Text]): Boolean
    var
        IsValid: Boolean;
    begin
        IsValid := true;

        if not Parameters.ContainsKey('CustomerNo') then begin
            ErrorMessages.Add('Missing required parameter: CustomerNo');
            IsValid := false;
        end;

        if not Parameters.ContainsKey('Amount') then begin
            ErrorMessages.Add('Missing required parameter: Amount');
            IsValid := false;
        end;

        exit(IsValid);
    end;

    local procedure ValidateBusinessRules(Parameters: Dictionary of [Text, Variant]; var ErrorMessages: List of [Text]): Boolean
    var
        Amount: Decimal;
        IsValid: Boolean;
    begin
        IsValid := true;

        if Parameters.Get('Amount', Amount) then begin
            if Amount <= 0 then begin
                ErrorMessages.Add('Amount must be greater than zero');
                IsValid := false;
            end;
        end;

        exit(IsValid);
    end;
}
```

This implementation shows how delegate operations enable flexible, extensible APIs in Business Central while maintaining clean separation of concerns and supporting runtime configuration of business logic.