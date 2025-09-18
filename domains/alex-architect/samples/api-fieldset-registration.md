# API Fieldset Registration Sample

## Overview
This sample demonstrates how to implement fieldset registration patterns for dynamic API field management and metadata-driven field selection.

## Basic Fieldset Registration

```al
codeunit 50100 "API Fieldset Registration"
{
    procedure RegisterCustomerFieldset()
    var
        FieldsetRegistry: Codeunit "Fieldset Registry";
        CustomerFieldset: Record "API Fieldset";
    begin
        // Register basic customer fieldset
        CustomerFieldset.Init();
        CustomerFieldset."Table ID" := Database::Customer;
        CustomerFieldset."Fieldset Name" := 'BASIC';
        CustomerFieldset."Field No." := Customer.FieldNo("No.");
        CustomerFieldset.Insert();

        CustomerFieldset."Field No." := Customer.FieldNo(Name);
        CustomerFieldset.Insert();

        CustomerFieldset."Field No." := Customer.FieldNo("Phone No.");
        CustomerFieldset.Insert();

        // Register extended customer fieldset
        CustomerFieldset."Fieldset Name" := 'EXTENDED';
        CustomerFieldset."Field No." := Customer.FieldNo("Credit Limit (LCY)");
        CustomerFieldset.Insert();

        CustomerFieldset."Field No." := Customer.FieldNo("Balance (LCY)");
        CustomerFieldset.Insert();
    end;
}
```

## Dynamic Field Selection

```al
codeunit 50101 "Dynamic Field Selection"
{
    procedure GetCustomerFields(FieldsetName: Text): List of [Integer]
    var
        APIFieldset: Record "API Fieldset";
        FieldList: List of [Integer];
    begin
        APIFieldset.SetRange("Table ID", Database::Customer);
        APIFieldset.SetRange("Fieldset Name", FieldsetName);
        if APIFieldset.FindSet() then
            repeat
                FieldList.Add(APIFieldset."Field No.");
            until APIFieldset.Next() = 0;

        exit(FieldList);
    end;

    procedure ApplyFieldset(var CustomerRec: Record Customer; FieldsetName: Text)
    var
        FieldList: List of [Integer];
        FieldNo: Integer;
    begin
        FieldList := GetCustomerFields(FieldsetName);
        CustomerRec.SetLoadFields();

        foreach FieldNo in FieldList do
            CustomerRec.AddLoadFields(FieldNo);
    end;
}
```

## Conditional Registration

```al
codeunit 50102 "Conditional Fieldset Reg"
{
    procedure RegisterConditionalFieldset(TableID: Integer; Condition: Text)
    var
        ConditionalFieldset: Record "Conditional API Fieldset";
    begin
        ConditionalFieldset.Init();
        ConditionalFieldset."Table ID" := TableID;
        ConditionalFieldset.Condition := Condition;
        ConditionalFieldset."Field Selection" := GetFieldsForCondition(TableID, Condition);
        ConditionalFieldset.Insert();
    end;

    local procedure GetFieldsForCondition(TableID: Integer; Condition: Text): Text
    var
        FieldSelection: Text;
    begin
        case Condition of
            'MINIMAL':
                FieldSelection := 'No.,Name';
            'STANDARD':
                FieldSelection := 'No.,Name,Phone No.,E-Mail';
            'COMPLETE':
                FieldSelection := '*';
        end;

        exit(FieldSelection);
    end;
}
```

## API Integration

```al
page 50100 "Customer API with Fieldsets"
{
    APIPublisher = 'contoso';
    APIGroup = 'sample';
    APIVersion = 'v1.0';
    EntityName = 'customer';
    EntitySetName = 'customers';
    SourceTable = Customer;

    trigger OnOpenPage()
    var
        FieldsetManager: Codeunit "Dynamic Field Selection";
        RequestedFieldset: Text;
    begin
        // Get fieldset from query parameter
        RequestedFieldset := GetFieldsetFromRequest();
        if RequestedFieldset <> '' then
            FieldsetManager.ApplyFieldset(Rec, RequestedFieldset);
    end;

    local procedure GetFieldsetFromRequest(): Text
    var
        RequestHeaders: HttpHeaders;
        FieldsetHeader: Text;
    begin
        // Extract fieldset preference from request
        if RequestHeaders.Contains('X-Fieldset') then begin
            RequestHeaders.GetValues('X-Fieldset', FieldsetHeader);
            exit(FieldsetHeader);
        end;

        exit('STANDARD');
    end;
}
```

## Metadata-Driven Configuration

```al
table 50100 "API Fieldset"
{
    fields
    {
        field(1; "Table ID"; Integer) { }
        field(2; "Fieldset Name"; Code[20]) { }
        field(3; "Field No."; Integer) { }
        field(4; "Field Name"; Text[100]) { }
        field(5; Required; Boolean) { }
        field(6; "Display Order"; Integer) { }
    }

    keys
    {
        key(PK; "Table ID", "Fieldset Name", "Field No.") { Clustered = true; }
    }
}
```

## Performance Considerations

```al
codeunit 50103 "Fieldset Performance Mgmt"
{
    var
        FieldsetCache: Dictionary of [Text, List of [Integer]];

    procedure GetCachedFieldset(TableID: Integer; FieldsetName: Text): List of [Integer]
    var
        CacheKey: Text;
        FieldList: List of [Integer];
    begin
        CacheKey := StrSubstNo('%1_%2', TableID, FieldsetName);

        if FieldsetCache.ContainsKey(CacheKey) then
            exit(FieldsetCache.Get(CacheKey));

        FieldList := LoadFieldsetFromDatabase(TableID, FieldsetName);
        FieldsetCache.Set(CacheKey, FieldList);

        exit(FieldList);
    end;

    procedure ClearCache()
    begin
        Clear(FieldsetCache);
    end;
}
```

This sample demonstrates comprehensive fieldset registration patterns including basic registration, dynamic selection, conditional logic, API integration, and performance optimization through caching.