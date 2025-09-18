# Event Bridge Pattern in AL - Code Examples

This sample demonstrates implementing event bridges for cross-module communication in Business Central.

## Basic Event Bridge Interface

```al
// Core event bridge interface
interface IEventBridge
{
    procedure PublishEvent(EventType: Text[50]; EventData: Dictionary of [Text, Variant]);
    procedure SubscribeToEvent(EventType: Text[50]; Handler: Interface IEventHandler);
    procedure RouteEvent(EventType: Text[50]; EventData: Dictionary of [Text, Variant]);
}
```

## Event Handler Interface

```al
// Event handler contract
interface IEventHandler
{
    procedure HandleEvent(EventData: Dictionary of [Text, Variant]): Boolean;
    procedure GetHandlerName(): Text[50];
    procedure CanHandle(EventType: Text[50]): Boolean;
}
```

## Central Event Bridge Implementation

```al
// Main event bridge implementation
codeunit 50200 "Central Event Bridge" implements IEventBridge
{
    var
        EventHandlers: Dictionary of [Text, List of [Interface IEventHandler]];
        EventLog: List of [Text];

    procedure PublishEvent(EventType: Text[50]; EventData: Dictionary of [Text, Variant])
    begin
        // Add event metadata
        EventData.Set('EventType', EventType);
        EventData.Set('Timestamp', CurrentDateTime);
        EventData.Set('Source', 'CentralBridge');

        // Log event
        LogEvent('PUBLISH', EventType, EventData);

        // Route to handlers
        RouteEvent(EventType, EventData);
    end;

    procedure SubscribeToEvent(EventType: Text[50]; Handler: Interface IEventHandler)
    var
        HandlerList: List of [Interface IEventHandler];
    begin
        if not EventHandlers.Get(EventType, HandlerList) then
            Clear(HandlerList);

        HandlerList.Add(Handler);
        EventHandlers.Set(EventType, HandlerList);

        LogEvent('SUBSCRIBE', EventType, Handler.GetHandlerName());
    end;

    procedure RouteEvent(EventType: Text[50]; EventData: Dictionary of [Text, Variant])
    var
        HandlerList: List of [Interface IEventHandler];
        Handler: Interface IEventHandler;
        ProcessingResult: Boolean;
    begin
        if not EventHandlers.Get(EventType, HandlerList) then
            exit;

        foreach Handler in HandlerList do begin
            if Handler.CanHandle(EventType) then begin
                ProcessingResult := Handler.HandleEvent(EventData);
                LogEventHandling(EventType, Handler.GetHandlerName(), ProcessingResult);
            end;
        end;
    end;

    local procedure LogEvent(Action: Text[20]; EventType: Text[50]; Details: Variant)
    var
        LogEntry: Text;
    begin
        LogEntry := StrSubstNo('%1: %2 - %3 at %4', Action, EventType, Details, CurrentDateTime);
        EventLog.Add(LogEntry);
    end;

    local procedure LogEventHandling(EventType: Text[50]; HandlerName: Text[50]; Success: Boolean)
    var
        Result: Text[20];
        LogEntry: Text;
    begin
        if Success then
            Result := 'SUCCESS'
        else
            Result := 'FAILED';

        LogEntry := StrSubstNo('HANDLE: %1 by %2 - %3', EventType, HandlerName, Result);
        EventLog.Add(LogEntry);
    end;
}
```

## Sales Event Handler Example

```al
// Sales-specific event handler
codeunit 50201 "Sales Event Handler" implements IEventHandler
{
    procedure HandleEvent(EventData: Dictionary of [Text, Variant]): Boolean
    var
        SalesHeader: Record "Sales Header";
        DocumentNo: Code[20];
        EventType: Text[50];
    begin
        if not EventData.Get('EventType', EventType) then
            exit(false);

        case EventType of
            'SALES_DOCUMENT_CREATED':
                exit(HandleSalesDocumentCreated(EventData));
            'SALES_DOCUMENT_POSTED':
                exit(HandleSalesDocumentPosted(EventData));
            'CUSTOMER_UPDATED':
                exit(HandleCustomerUpdated(EventData));
            else
                exit(false);
        end;
    end;

    procedure GetHandlerName(): Text[50]
    begin
        exit('SalesEventHandler');
    end;

    procedure CanHandle(EventType: Text[50]): Boolean
    begin
        exit(EventType in ['SALES_DOCUMENT_CREATED', 'SALES_DOCUMENT_POSTED', 'CUSTOMER_UPDATED']);
    end;

    local procedure HandleSalesDocumentCreated(EventData: Dictionary of [Text, Variant]): Boolean
    var
        DocumentNo: Code[20];
        CustomerNo: Code[20];
    begin
        if not EventData.Get('DocumentNo', DocumentNo) then
            exit(false);

        if not EventData.Get('CustomerNo', CustomerNo) then
            exit(false);

        // Process sales document creation
        Message('Sales document %1 created for customer %2', DocumentNo, CustomerNo);

        // Trigger additional business logic
        TriggerInventoryCheck(DocumentNo);
        UpdateCustomerStatistics(CustomerNo);

        exit(true);
    end;

    local procedure HandleSalesDocumentPosted(EventData: Dictionary of [Text, Variant]): Boolean
    var
        DocumentNo: Code[20];
        PostedDocumentNo: Code[20];
    begin
        if not EventData.Get('DocumentNo', DocumentNo) then
            exit(false);

        if not EventData.Get('PostedDocumentNo', PostedDocumentNo) then
            exit(false);

        // Process document posting
        UpdateCustomerLastSaleDate(EventData);
        TriggerReorderPointCheck(DocumentNo);

        exit(true);
    end;

    local procedure HandleCustomerUpdated(EventData: Dictionary of [Text, Variant]): Boolean
    var
        CustomerNo: Code[20];
        FieldChanged: Text[50];
    begin
        if not EventData.Get('CustomerNo', CustomerNo) then
            exit(false);

        EventData.Get('FieldChanged', FieldChanged);

        // React to customer changes
        if FieldChanged = 'Credit Limit' then
            ReviewPendingOrders(CustomerNo);

        exit(true);
    end;

    local procedure TriggerInventoryCheck(DocumentNo: Code[20])
    begin
        // Implementation for inventory checking
    end;

    local procedure UpdateCustomerStatistics(CustomerNo: Code[20])
    begin
        // Implementation for customer statistics update
    end;

    local procedure UpdateCustomerLastSaleDate(EventData: Dictionary of [Text, Variant])
    begin
        // Implementation for last sale date update
    end;

    local procedure TriggerReorderPointCheck(DocumentNo: Code[20])
    begin
        // Implementation for reorder point checking
    end;

    local procedure ReviewPendingOrders(CustomerNo: Code[20])
    begin
        // Implementation for pending order review
    end;
}
```

## Cross-Module Event Publisher

```al
// Sales document event publisher
codeunit 50202 "Sales Document Events"
{
    var
        EventBridge: Codeunit "Central Event Bridge";

    [EventSubscriber(ObjectType::Table, Database::"Sales Header", 'OnAfterInsertEvent', '', false, false)]
    local procedure OnSalesHeaderInsert(var Rec: Record "Sales Header")
    var
        EventData: Dictionary of [Text, Variant];
    begin
        // Prepare event data
        EventData.Set('DocumentNo', Rec."No.");
        EventData.Set('CustomerNo', Rec."Sell-to Customer No.");
        EventData.Set('DocumentType', Format(Rec."Document Type"));
        EventData.Set('Amount', Rec."Amount Including VAT");

        // Publish through bridge
        EventBridge.PublishEvent('SALES_DOCUMENT_CREATED', EventData);
    end;

    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", 'OnAfterSalesInvHeaderInsert', '', false, false)]
    local procedure OnSalesInvoicePosted(var SalesInvHeader: Record "Sales Invoice Header"; SalesHeader: Record "Sales Header")
    var
        EventData: Dictionary of [Text, Variant];
    begin
        // Prepare posting event data
        EventData.Set('DocumentNo', SalesHeader."No.");
        EventData.Set('PostedDocumentNo', SalesInvHeader."No.");
        EventData.Set('CustomerNo', SalesInvHeader."Sell-to Customer No.");
        EventData.Set('PostingDate', SalesInvHeader."Posting Date");
        EventData.Set('Amount', SalesInvHeader."Amount Including VAT");

        // Publish posting event
        EventBridge.PublishEvent('SALES_DOCUMENT_POSTED', EventData);
    end;
}
```

## Event Transformation Bridge

```al
// Bridge that transforms events between different formats
codeunit 50203 "Event Transformation Bridge" implements IEventBridge
{
    var
        CentralBridge: Codeunit "Central Event Bridge";

    procedure PublishEvent(EventType: Text[50]; EventData: Dictionary of [Text, Variant])
    var
        TransformedData: Dictionary of [Text, Variant];
    begin
        // Transform event data before publishing
        TransformedData := TransformEventData(EventType, EventData);

        // Publish transformed event
        CentralBridge.PublishEvent(EventType, TransformedData);
    end;

    procedure SubscribeToEvent(EventType: Text[50]; Handler: Interface IEventHandler)
    begin
        // Delegate subscription to central bridge
        CentralBridge.SubscribeToEvent(EventType, Handler);
    end;

    procedure RouteEvent(EventType: Text[50]; EventData: Dictionary of [Text, Variant])
    begin
        // Delegate routing to central bridge
        CentralBridge.RouteEvent(EventType, EventData);
    end;

    local procedure TransformEventData(EventType: Text[50]; OriginalData: Dictionary of [Text, Variant]): Dictionary of [Text, Variant]
    var
        TransformedData: Dictionary of [Text, Variant];
        Key: Text;
        Value: Variant;
    begin
        // Copy original data
        foreach Key in OriginalData.Keys do begin
            OriginalData.Get(Key, Value);
            TransformedData.Set(Key, Value);
        end;

        // Add transformation metadata
        TransformedData.Set('TransformedBy', 'EventTransformationBridge');
        TransformedData.Set('TransformationTime', CurrentDateTime);

        // Apply specific transformations
        case EventType of
            'SALES_DOCUMENT_CREATED':
                TransformSalesDocumentEvent(TransformedData);
            'CUSTOMER_UPDATED':
                TransformCustomerEvent(TransformedData);
        end;

        exit(TransformedData);
    end;

    local procedure TransformSalesDocumentEvent(var EventData: Dictionary of [Text, Variant])
    var
        Amount: Decimal;
        Currency: Code[10];
    begin
        // Add computed fields
        if EventData.Get('Amount', Amount) then begin
            EventData.Set('AmountCategory', GetAmountCategory(Amount));
            EventData.Set('RequiresApproval', Amount > 10000);
        end;

        // Add additional context
        EventData.Set('BusinessArea', 'Sales');
        EventData.Set('Priority', 'Standard');
    end;

    local procedure TransformCustomerEvent(var EventData: Dictionary of [Text, Variant])
    begin
        // Add customer-specific transformations
        EventData.Set('BusinessArea', 'Customer Management');
        EventData.Set('RequiresReview', true);
    end;

    local procedure GetAmountCategory(Amount: Decimal): Text[20]
    begin
        case true of
            Amount <= 1000:
                exit('Small');
            Amount <= 10000:
                exit('Medium');
            else
                exit('Large');
        end;
    end;
}
```

## Event Bridge Setup

```al
// Setup and initialization for event bridge system
codeunit 50204 "Event Bridge Setup"
{
    var
        CentralBridge: Codeunit "Central Event Bridge";
        SalesHandler: Codeunit "Sales Event Handler";

    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Company-Initialize", 'OnCompanyInitialize', '', false, false)]
    local procedure OnCompanyInitialize()
    begin
        SetupEventBridge();
    end;

    local procedure SetupEventBridge()
    begin
        // Register event handlers
        CentralBridge.SubscribeToEvent('SALES_DOCUMENT_CREATED', SalesHandler);
        CentralBridge.SubscribeToEvent('SALES_DOCUMENT_POSTED', SalesHandler);
        CentralBridge.SubscribeToEvent('CUSTOMER_UPDATED', SalesHandler);

        // Additional setup as needed
        Message('Event bridge system initialized successfully');
    end;

    procedure GetEventBridge(): Codeunit "Central Event Bridge"
    begin
        exit(CentralBridge);
    end;
}
```

This implementation provides a comprehensive event bridge system that enables loose coupling between modules while supporting event transformation, routing, and comprehensive logging for debugging and monitoring.