# AL Begin/End Block Structure - Code Examples

## Object Trigger Structure

### Table Trigger Pattern
```al
table 50100 "Sales Order Processing"
{
    fields
    {
        field(1; "Order No."; Code[20])
        {
            Caption = 'Order No.';
        }
        field(2; "Customer No."; Code[20])
        {
            Caption = 'Customer No.';
            TableRelation = Customer;

            trigger OnValidate()
            begin
                if "Customer No." <> '' then begin
                    Customer.Get("Customer No.");
                    "Customer Name" := Customer.Name;
                    ValidateCustomerCredit();
                end;
            end;
        }
    }

    var
        Customer: Record Customer;

    trigger OnInsert()
    begin
        TestField("Customer No.");

        if "Order No." = '' then begin
            NoSeriesManagement.InitSeries(GetNoSeriesCode(), xRec."No. Series", 0D, "Order No.", "No. Series");
        end;
    end;

    trigger OnModify()
    begin
        if xRec."Customer No." <> "Customer No." then begin
            if OrderLineExists() then begin
                if not Confirm('Customer changed. Update order lines?') then begin
                    Error('Customer change cancelled');
                end;
                UpdateOrderLines();
            end;
        end;
    end;
}
```

### Page Trigger Structure
```al
page 50100 "Sales Order Card"
{
    PageType = Card;
    SourceTable = "Sales Order Processing";

    layout
    {
        area(Content)
        {
            group(General)
            {
                field("Order No."; "Order No.")
                {
                    ApplicationArea = All;

                    trigger OnAssistEdit()
                    begin
                        if AssistEdit(xRec) then begin
                            CurrPage.Update();
                        end;
                    end;
                }
            }
        }
    }

    actions
    {
        area(Processing)
        {
            action(ProcessOrder)
            {
                Caption = 'Process Order';

                trigger OnAction()
                begin
                    if ValidateOrderData() then begin
                        if ConfirmProcessing() then begin
                            ProcessOrderLines();
                            SendConfirmationEmail();
                            Message('Order processed successfully');
                        end else begin
                            Message('Order processing cancelled');
                        end;
                    end else begin
                        Error('Order validation failed');
                    end;
                end;
            }
        }
    }

    trigger OnOpenPage()
    begin
        SetDefaultFilters();
        UpdateVisibility();

        if IsNewRecord() then begin
            InitializeNewOrder();
        end;
    end;
}
```

## Procedure Organization Patterns

```al
codeunit 50100 "Order Management"
{
    procedure CreateSalesOrder(CustomerNo: Code[20]; OrderDate: Date): Code[20]
    var
        SalesHeader: Record "Sales Header";
        OrderNo: Code[20];
    begin
        OrderNo := GetNextOrderNumber();

        SalesHeader.Init();
        SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
        SalesHeader."No." := OrderNo;
        SalesHeader."Sell-to Customer No." := CustomerNo;
        SalesHeader."Order Date" := OrderDate;
        SalesHeader.Insert(true);

        exit(OrderNo);
    end;

    procedure ProcessBulkOrders(var TempOrderBuffer: Record "Order Buffer" temporary)
    var
        OrderNo: Code[20];
        ProcessedCount: Integer;
        ErrorCount: Integer;
    begin
        if TempOrderBuffer.IsEmpty() then begin
            Message('No orders to process');
            exit;
        end;

        TempOrderBuffer.FindSet();
        repeat
            if ValidateOrderBuffer(TempOrderBuffer) then begin
                if CreateOrderFromBuffer(TempOrderBuffer, OrderNo) then begin
                    ProcessedCount += 1;
                    TempOrderBuffer."Status" := TempOrderBuffer."Status"::Processed;
                    TempOrderBuffer."Order No." := OrderNo;
                end else begin
                    ErrorCount += 1;
                    TempOrderBuffer."Status" := TempOrderBuffer."Status"::Error;
                end;
            end else begin
                ErrorCount += 1;
                TempOrderBuffer."Status" := TempOrderBuffer."Status"::"Validation Error";
            end;

            TempOrderBuffer.Modify();
        until TempOrderBuffer.Next() = 0;

        Message('Processing complete. Processed: %1, Errors: %2', ProcessedCount, ErrorCount);
    end;
}
```

## Control Flow Nesting Patterns

```al
codeunit 50101 "Advanced Control Flow"
{
    procedure DetermineShippingMethod(SalesHeader: Record "Sales Header"): Code[10]
    begin
        if SalesHeader."Sell-to Country/Region Code" = 'US' then begin
            if SalesHeader."Order Date" = Today() then begin
                if SalesHeader.Amount > 1000 then begin
                    exit('RUSH-PREMIUM');
                end else begin
                    exit('RUSH-STANDARD');
                end;
            end else begin
                if SalesHeader.Amount > 500 then begin
                    exit('DOMESTIC-FREE');
                end else begin
                    exit('DOMESTIC-PAID');
                end;
            end;
        end else begin
            if SalesHeader.Amount > 2000 then begin
                exit('INTL-EXPRESS');
            end else begin
                if GetShippingDistance(SalesHeader."Sell-to Country/Region Code") <= 5000 then begin
                    exit('INTL-STANDARD');
                end else begin
                    exit('INTL-ECONOMY');
                end;
            end;
        end;
    end;

    procedure ProcessOrderByType(OrderType: Enum "Order Type")
    begin
        case OrderType of
            OrderType::Standard:
                begin
                    ValidateInventory();
                    CreateShipment();
                    SendConfirmation();
                end;
            OrderType::DropShip:
                begin
                    ValidateVendor();
                    CreatePurchaseOrder();
                    NotifyCustomer();
                end;
            OrderType::Transfer:
                begin
                    ValidateLocations();
                    CreateTransferOrder();
                    UpdateInventory();
                end;
            else begin
                Error('Unsupported order type: %1', OrderType);
            end;
        end;
    end;
}
```

## Variable Scope Examples

```al
codeunit 50102 "Variable Scope Examples"
{
    var
        GlobalCounter: Integer;
        GlobalCustomer: Record Customer;

    procedure DemonstrateVariableScope()
    var
        LocalCounter: Integer;
        Customer: Record Customer;
        i: Integer;
    begin
        LocalCounter := 0;

        for i := 1 to 10 do begin
            LocalCounter += 1;
            GlobalCounter += 1;

            if i mod 2 = 0 then begin
                Customer.SetRange("No.", Format(i));
                if Customer.FindFirst() then begin
                    ProcessCustomer(Customer);
                end;
            end;
        end;

        Message('Local: %1, Global: %2', LocalCounter, GlobalCounter);
    end;

    procedure ProcessCustomerOrders(CustomerNo: Code[20]; ProcessDate: Date)
    var
        SalesHeader: Record "Sales Header";
        OrderCount: Integer;
        TotalAmount: Decimal;
    begin
        OrderCount := 0;
        TotalAmount := 0;

        SalesHeader.SetRange("Sell-to Customer No.", CustomerNo);
        SalesHeader.SetRange("Order Date", ProcessDate);

        if SalesHeader.FindSet() then begin
            repeat
                OrderCount += 1;
                SalesHeader.CalcFields(Amount);
                TotalAmount += SalesHeader.Amount;

                if SalesHeader.Amount > 1000 then begin
                    ApplyVIPProcessing(SalesHeader);
                end;

            until SalesHeader.Next() = 0;
        end;

        UpdateCustomerStatistics(CustomerNo, OrderCount, TotalAmount);
    end;
}
```

## Exception Handling Patterns

```al
codeunit 50103 "Exception Handling Examples"
{
    procedure SafeOrderProcessing(OrderNo: Code[20]): Boolean
    var
        SalesHeader: Record "Sales Header";
        ProcessingResult: Boolean;
    begin
        ProcessingResult := false;

        if SalesHeader.Get(SalesHeader."Document Type"::Order, OrderNo) then begin
            if ValidateOrderForProcessing(SalesHeader) then begin
                if ProcessOrderSafely(SalesHeader) then begin
                    ProcessingResult := true;
                end;
            end;
        end;

        exit(ProcessingResult);
    end;

    local procedure ProcessOrderSafely(var SalesHeader: Record "Sales Header"): Boolean
    begin
        if not Codeunit.Run(Codeunit::"Sales-Post", SalesHeader) then begin
            LogProcessingError(SalesHeader."No.", GetLastErrorText());
            ClearLastError();
            exit(false);
        end;

        exit(true);
    end;

    procedure BatchProcessOrders(var TempOrderList: Record "Order Processing Queue" temporary)
    var
        ProcessedCount: Integer;
        ErrorCount: Integer;
        CurrentOrder: Code[20];
    begin
        ProcessedCount := 0;
        ErrorCount := 0;

        if TempOrderList.FindSet() then begin
            repeat
                CurrentOrder := TempOrderList."Order No.";

                if ProcessSingleOrder(CurrentOrder) then begin
                    ProcessedCount += 1;
                    TempOrderList.Status := TempOrderList.Status::Completed;
                end else begin
                    ErrorCount += 1;
                    TempOrderList.Status := TempOrderList.Status::Error;
                    TempOrderList."Error Message" := GetLastErrorText();
                    ClearLastError();
                end;

                TempOrderList.Modify();

            until TempOrderList.Next() = 0;
        end;

        Message('Batch processing complete.\Processed: %1\Errors: %2', ProcessedCount, ErrorCount);
    end;
}
```

## Anti-Pattern Examples

```al
// ❌ ANTI-PATTERN: Poor block structure
procedure BadExample()
var
    Customer: Record Customer;
begin
    // Missing begin/end for multi-statement block
    if Customer.FindFirst() then
        Customer.CalcFields(Balance);
        UpdateCustomerStatus(Customer);  // This always executes!

    // Inconsistent block structure
    repeat
        ProcessCustomer();
    until Customer.Next() = 0;  // Should be in begin/end block
end;

// ❌ ANTI-PATTERN: Poor variable scope
procedure BadVariableScope()
begin
    if SomeCondition then begin
        Customer: Record Customer;  // SYNTAX ERROR - can't declare here
        Customer.FindFirst();
    end;
end;

// ✅ CORRECT: Proper block structure
procedure GoodExample()
var
    Customer: Record Customer;
begin
    if Customer.FindFirst() then begin
        Customer.CalcFields(Balance);
        UpdateCustomerStatus(Customer);
    end;

    if Customer.FindSet() then begin
        repeat
            ProcessCustomer();
        until Customer.Next() = 0;
    end;
end;
```

## Best Practices Summary

### Block Structure Guidelines
1. **Always use begin/end** for multi-statement blocks
2. **Consistent indentation** - use 4 spaces per level
3. **Clear nesting** - avoid deeply nested structures
4. **Logical grouping** - group related statements
5. **Single responsibility** - each block has clear purpose

### Variable Scope Best Practices
1. **Declare at procedure level** for maximum flexibility
2. **Initialize variables** before use
3. **Use meaningful names** indicating scope and purpose
4. **Minimize global variables** - prefer parameters

### Error Handling Standards
1. **Check return values** from system functions
2. **Use ClearLastError()** after handling errors
3. **Provide meaningful messages** with context
4. **Implement graceful degradation** where possible
5. **Log errors appropriately** for troubleshooting