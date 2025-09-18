# Type-Safe Operations Sample

## Overview
This sample demonstrates type-safe operation patterns in AL including generic procedures, interface-based type safety, and compile-time type validation techniques.

## Generic Type-Safe Procedures

```al
codeunit 50400 "Type Safe Operations"
{
    procedure SafeGetRecord<T>(RecordNo: Code[20]; var TargetRecord: Record T): Boolean
    begin
        TargetRecord.SetRange("No.", RecordNo);
        exit(TargetRecord.FindFirst());
    end;

    procedure SafeUpdateField<T>(var TargetRecord: Record T; FieldNo: Integer; NewValue: Variant): Boolean
    var
        RecordRef: RecordRef;
        FieldRef: FieldRef;
    begin
        RecordRef.GetTable(TargetRecord);

        if not RecordRef.FieldExist(FieldNo) then
            exit(false);

        FieldRef := RecordRef.Field(FieldNo);

        if not ValidateFieldType(FieldRef, NewValue) then
            exit(false);

        FieldRef.Value := NewValue;
        RecordRef.SetTable(TargetRecord);

        exit(TargetRecord.Modify(true));
    end;

    local procedure ValidateFieldType(FieldRef: FieldRef; NewValue: Variant): Boolean
    begin
        case FieldRef.Type of
            FieldType::Code:
                exit(NewValue.IsCode);
            FieldType::Text:
                exit(NewValue.IsText);
            FieldType::Integer:
                exit(NewValue.IsInteger);
            FieldType::Decimal:
                exit(NewValue.IsDecimal);
            FieldType::Boolean:
                exit(NewValue.IsBoolean);
            FieldType::Date:
                exit(NewValue.IsDate);
            FieldType::DateTime:
                exit(NewValue.IsDateTime);
            else
                exit(false);
        end;
    end;
}
```

## Interface-Based Type Safety

```al
interface "Type Safe Validator"
{
    procedure Validate(Value: Variant): Boolean;
    procedure GetValidationMessage(): Text;
}

codeunit 50401 "Customer Validator" implements "Type Safe Validator"
{
    procedure Validate(Value: Variant): Boolean
    var
        Customer: Record Customer;
    begin
        if not Value.IsRecord then
            exit(false);

        Customer := Value;
        exit(ValidateCustomer(Customer));
    end;

    procedure GetValidationMessage(): Text
    begin
        exit('Customer validation failed');
    end;

    local procedure ValidateCustomer(Customer: Record Customer): Boolean
    begin
        if Customer."No." = '' then
            exit(false);

        if Customer.Name = '' then
            exit(false);

        if Customer.Blocked <> Customer.Blocked::" " then
            exit(false);

        exit(true);
    end;
}

codeunit 50402 "Item Validator" implements "Type Safe Validator"
{
    procedure Validate(Value: Variant): Boolean
    var
        Item: Record Item;
    begin
        if not Value.IsRecord then
            exit(false);

        Item := Value;
        exit(ValidateItem(Item));
    end;

    procedure GetValidationMessage(): Text
    begin
        exit('Item validation failed');
    end;

    local procedure ValidateItem(Item: Record Item): Boolean
    begin
        if Item."No." = '' then
            exit(false);

        if Item.Description = '' then
            exit(false);

        if Item.Type = Item.Type::" " then
            exit(false);

        exit(true);
    end;
}
```

## Type-Safe Data Processing

```al
codeunit 50403 "Type Safe Data Processor"
{
    procedure ProcessTypedData<T>(var DataRecord: Record T; Processor: Interface "Data Processor"): Boolean
    var
        RecordVariant: Variant;
    begin
        RecordVariant := DataRecord;

        if not Processor.CanProcess(RecordVariant) then
            exit(false);

        if not Processor.Process(RecordVariant) then
            exit(false);

        DataRecord := RecordVariant;
        exit(true);
    end;

    procedure BatchProcessRecords<T>(var RecordSet: Record T; Processor: Interface "Data Processor"): Integer
    var
        ProcessedCount: Integer;
        CurrentRecord: Record T;
    begin
        RecordSet.FindSet();
        repeat
            CurrentRecord := RecordSet;
            if ProcessTypedData(CurrentRecord, Processor) then
                ProcessedCount += 1;
        until RecordSet.Next() = 0;

        exit(ProcessedCount);
    end;
}

interface "Data Processor"
{
    procedure CanProcess(Data: Variant): Boolean;
    procedure Process(var Data: Variant): Boolean;
    procedure GetProcessorName(): Text;
}
```

## Type-Safe Configuration Pattern

```al
codeunit 50404 "Type Safe Config Manager"
{
    procedure GetConfigValue<T>(ConfigKey: Code[50]): T
    var
        ConfigRecord: Record "Type Safe Configuration";
        ResultValue: T;
        ConfigVariant: Variant;
    begin
        if not ConfigRecord.Get(ConfigKey) then
            Error('Configuration key %1 not found', ConfigKey);

        ConfigVariant := ConfigRecord.Value;

        if not TryConvertToType<T>(ConfigVariant, ResultValue) then
            Error('Cannot convert configuration value to requested type for key %1', ConfigKey);

        exit(ResultValue);
    end;

    procedure SetConfigValue<T>(ConfigKey: Code[50]; Value: T): Boolean
    var
        ConfigRecord: Record "Type Safe Configuration";
        ValueVariant: Variant;
    begin
        ValueVariant := Value;

        if ConfigRecord.Get(ConfigKey) then
            ConfigRecord.Modify()
        else begin
            ConfigRecord.Init();
            ConfigRecord."Config Key" := ConfigKey;
            ConfigRecord.Insert();
        end;

        ConfigRecord.Value := ValueVariant;
        ConfigRecord."Data Type" := GetVariantType(ValueVariant);

        exit(ConfigRecord.Modify(true));
    end;

    [TryFunction]
    local procedure TryConvertToType<T>(SourceVariant: Variant; var TargetValue: T)
    var
        TempVariant: Variant;
    begin
        TempVariant := SourceVariant;
        TargetValue := TempVariant;
    end;

    local procedure GetVariantType(Value: Variant): Enum "Configuration Data Type"
    begin
        case true of
            Value.IsBoolean:
                exit("Configuration Data Type"::Boolean);
            Value.IsInteger:
                exit("Configuration Data Type"::Integer);
            Value.IsDecimal:
                exit("Configuration Data Type"::Decimal);
            Value.IsText:
                exit("Configuration Data Type"::Text);
            Value.IsCode:
                exit("Configuration Data Type"::Code);
            Value.IsDate:
                exit("Configuration Data Type"::Date);
            Value.IsDateTime:
                exit("Configuration Data Type"::DateTime);
            else
                exit("Configuration Data Type"::Variant);
        end;
    end;
}
```

## Type-Safe Event Handling

```al
codeunit 50405 "Type Safe Event Manager"
{
    procedure RegisterEventHandler<T>(EventName: Code[50]; Handler: Interface "Typed Event Handler")
    var
        EventRegistration: Record "Event Handler Registration";
    begin
        EventRegistration.Init();
        EventRegistration."Event Name" := EventName;
        EventRegistration."Handler Type" := GetTypeName<T>();
        EventRegistration."Handler Interface" := Handler;
        EventRegistration.Active := true;
        EventRegistration.Insert(true);
    end;

    procedure RaiseTypedEvent<T>(EventName: Code[50]; EventData: T): Boolean
    var
        EventRegistration: Record "Event Handler Registration";
        Handler: Interface "Typed Event Handler";
        EventVariant: Variant;
        HandlerExecuted: Boolean;
    begin
        EventVariant := EventData;

        EventRegistration.SetRange("Event Name", EventName);
        EventRegistration.SetRange("Handler Type", GetTypeName<T>());
        EventRegistration.SetRange(Active, true);

        if EventRegistration.FindSet() then
            repeat
                Handler := EventRegistration."Handler Interface";
                if Handler.HandleEvent(EventVariant) then
                    HandlerExecuted := true;
            until EventRegistration.Next() = 0;

        exit(HandlerExecuted);
    end;

    local procedure GetTypeName<T>(): Text
    var
        TypeHelper: Codeunit "Type Helper";
        DummyValue: T;
        DummyVariant: Variant;
    begin
        DummyVariant := DummyValue;
        exit(TypeHelper.GetObjectName(DummyVariant));
    end;
}

interface "Typed Event Handler"
{
    procedure HandleEvent(EventData: Variant): Boolean;
    procedure GetHandlerName(): Text;
}
```

## Type-Safe Collection Operations

```al
codeunit 50406 "Type Safe Collections"
{
    procedure CreateTypedList<T>(): List of [T]
    var
        TypedList: List of [T];
    begin
        exit(TypedList);
    end;

    procedure AddToTypedList<T>(var TypedList: List of [T]; Item: T): Boolean
    begin
        TypedList.Add(Item);
        exit(true);
    end;

    procedure FilterTypedList<T>(var TypedList: List of [T]; FilterFunc: Interface "List Filter"): List of [T]
    var
        FilteredList: List of [T];
        Item: T;
        ItemVariant: Variant;
    begin
        foreach Item in TypedList do begin
            ItemVariant := Item;
            if FilterFunc.ShouldInclude(ItemVariant) then
                FilteredList.Add(Item);
        end;

        exit(FilteredList);
    end;

    procedure TransformTypedList<TSource, TTarget>(SourceList: List of [TSource]; TransformFunc: Interface "List Transform"): List of [TTarget]
    var
        TargetList: List of [TTarget];
        SourceItem: TSource;
        SourceVariant: Variant;
        TargetVariant: Variant;
        TargetItem: TTarget;
    begin
        foreach SourceItem in SourceList do begin
            SourceVariant := SourceItem;
            if TransformFunc.Transform(SourceVariant, TargetVariant) then begin
                TargetItem := TargetVariant;
                TargetList.Add(TargetItem);
            end;
        end;

        exit(TargetList);
    end;
}

interface "List Filter"
{
    procedure ShouldInclude(Item: Variant): Boolean;
}

interface "List Transform"
{
    procedure Transform(Source: Variant; var Target: Variant): Boolean;
}
```

## Type-Safe Error Handling

```al
codeunit 50407 "Type Safe Error Handler"
{
    procedure TryOperation<TInput, TOutput>(Input: TInput; Operation: Interface "Typed Operation"; var Output: TOutput): Boolean
    var
        InputVariant: Variant;
        OutputVariant: Variant;
        OperationResult: Boolean;
    begin
        InputVariant := Input;

        if not Operation.Validate(InputVariant) then
            exit(false);

        OperationResult := Operation.Execute(InputVariant, OutputVariant);

        if OperationResult then
            Output := OutputVariant;

        exit(OperationResult);
    end;

    procedure ExecuteWithFallback<T>(PrimaryOperation: Interface "Typed Operation"; FallbackOperation: Interface "Typed Operation"; Input: T): T
    var
        Output: T;
    begin
        if TryOperation(Input, PrimaryOperation, Output) then
            exit(Output);

        if TryOperation(Input, FallbackOperation, Output) then
            exit(Output);

        Error('All operations failed for input type %1', GetTypeName<T>());
    end;
}

interface "Typed Operation"
{
    procedure Validate(Input: Variant): Boolean;
    procedure Execute(Input: Variant; var Output: Variant): Boolean;
    procedure GetOperationName(): Text;
}
```

This sample demonstrates comprehensive type-safe operation patterns including generic procedures, interface-based validation, typed collections, configuration management, event handling, and error handling with compile-time type safety.