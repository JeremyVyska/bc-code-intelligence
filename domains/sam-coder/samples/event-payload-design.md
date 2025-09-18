# Event Payload Design Patterns Sample

## Overview
This sample demonstrates effective event payload design patterns including structured payloads, versioning strategies, and performance optimization techniques.

## Basic Event Payload Structure

```al
table 50300 "Event Payload Base"
{
    fields
    {
        field(1; "Event ID"; Guid) { }
        field(2; "Event Type"; Code[50]) { }
        field(3; "Source System"; Code[20]) { }
        field(4; "Event Version"; Code[10]) { }
        field(5; "Timestamp"; DateTime) { }
        field(6; "Correlation ID"; Guid) { }
        field(7; "Payload Data"; Blob) { }
        field(8; "Payload Schema"; Text[250]) { }
    }
}
```

## Structured Payload Design

```al
codeunit 50300 "Customer Event Payload Builder"
{
    procedure BuildCustomerCreatedPayload(Customer: Record Customer): JsonObject
    var
        Payload: JsonObject;
        CustomerData: JsonObject;
        EventMetadata: JsonObject;
    begin
        // Event metadata
        EventMetadata.Add('eventId', CreateGuid());
        EventMetadata.Add('eventType', 'customer.created');
        EventMetadata.Add('version', '1.0');
        EventMetadata.Add('timestamp', CurrentDateTime);
        EventMetadata.Add('source', 'business-central');

        // Customer data payload
        CustomerData.Add('customerNo', Customer."No.");
        CustomerData.Add('name', Customer.Name);
        CustomerData.Add('contactName', Customer.Contact);
        CustomerData.Add('phoneNo', Customer."Phone No.");
        CustomerData.Add('email', Customer."E-Mail");
        CustomerData.Add('blocked', Customer.Blocked);
        CustomerData.Add('creditLimit', Customer."Credit Limit (LCY)");

        // Address information
        AddAddressToPayload(CustomerData, Customer);

        // Combine metadata and data
        Payload.Add('metadata', EventMetadata);
        Payload.Add('data', CustomerData);

        exit(Payload);
    end;

    local procedure AddAddressToPayload(var CustomerData: JsonObject; Customer: Record Customer)
    var
        Address: JsonObject;
    begin
        Address.Add('address1', Customer.Address);
        Address.Add('address2', Customer."Address 2");
        Address.Add('city', Customer.City);
        Address.Add('postCode', Customer."Post Code");
        Address.Add('countryRegionCode', Customer."Country/Region Code");

        CustomerData.Add('address', Address);
    end;
}
```

## Versioned Payload Pattern

```al
codeunit 50301 "Versioned Payload Manager"
{
    procedure CreateVersionedPayload(EventType: Code[50]; Version: Code[10]; Data: Variant): JsonObject
    var
        Payload: JsonObject;
        VersionHandler: Interface "Payload Version Handler";
    begin
        VersionHandler := GetVersionHandler(EventType, Version);
        Payload := VersionHandler.BuildPayload(Data);

        // Add version compatibility information
        AddVersionCompatibility(Payload, EventType, Version);

        exit(Payload);
    end;

    local procedure GetVersionHandler(EventType: Code[50]; Version: Code[10]): Interface "Payload Version Handler"
    var
        V1Handler: Codeunit "Customer Event V1 Handler";
        V2Handler: Codeunit "Customer Event V2 Handler";
        V3Handler: Codeunit "Customer Event V3 Handler";
    begin
        case EventType of
            'customer.created',
            'customer.updated':
                case Version of
                    '1.0': exit(V1Handler);
                    '2.0': exit(V2Handler);
                    '3.0': exit(V3Handler);
                end;
        end;

        Error('Unsupported event type %1 version %2', EventType, Version);
    end;

    local procedure AddVersionCompatibility(var Payload: JsonObject; EventType: Code[50]; Version: Code[10])
    var
        CompatibilityInfo: JsonObject;
        SupportedVersions: JsonArray;
    begin
        GetSupportedVersions(EventType, SupportedVersions);

        CompatibilityInfo.Add('currentVersion', Version);
        CompatibilityInfo.Add('supportedVersions', SupportedVersions);
        CompatibilityInfo.Add('deprecationNotice', GetDeprecationNotice(EventType, Version));

        Payload.Add('versionInfo', CompatibilityInfo);
    end;
}
```

## Optimized Payload Pattern

```al
codeunit 50302 "Optimized Payload Builder"
{
    procedure BuildMinimalPayload(Customer: Record Customer; FieldsToInclude: List of [Text]): JsonObject
    var
        Payload: JsonObject;
        CustomerData: JsonObject;
        FieldName: Text;
    begin
        // Build metadata with minimal footprint
        AddMinimalMetadata(Payload);

        // Include only requested fields
        foreach FieldName in FieldsToInclude do
            AddFieldToPayload(CustomerData, Customer, FieldName);

        Payload.Add('data', CustomerData);
        exit(Payload);
    end;

    procedure BuildCompressedPayload(Customer: Record Customer): JsonObject
    var
        FullPayload: JsonObject;
        CompressedPayload: JsonObject;
        PayloadText: Text;
        CompressedText: Text;
    begin
        // Build full payload first
        FullPayload := BuildFullCustomerPayload(Customer);
        FullPayload.WriteTo(PayloadText);

        // Apply compression for large payloads
        if StrLen(PayloadText) > 1000 then begin
            CompressedText := CompressPayloadData(PayloadText);
            CompressedPayload.Add('compressed', true);
            CompressedPayload.Add('algorithm', 'gzip');
            CompressedPayload.Add('data', CompressedText);
            exit(CompressedPayload);
        end;

        exit(FullPayload);
    end;

    local procedure CompressPayloadData(PayloadText: Text): Text
    var
        TempBlob: Codeunit "Temp Blob";
        InStream: InStream;
        OutStream: OutStream;
        CompressedText: Text;
    begin
        // Simplified compression example
        TempBlob.CreateOutStream(OutStream);
        OutStream.WriteText(PayloadText);
        TempBlob.CreateInStream(InStream);

        // Apply compression algorithm here
        InStream.ReadText(CompressedText);

        exit(CompressedText);
    end;
}
```

## Hierarchical Payload Structure

```al
codeunit 50303 "Hierarchical Payload Builder"
{
    procedure BuildSalesOrderPayload(SalesHeader: Record "Sales Header"): JsonObject
    var
        Payload: JsonObject;
        OrderData: JsonObject;
        LinesArray: JsonArray;
        SalesLine: Record "Sales Line";
    begin
        // Header information
        OrderData.Add('orderNo', SalesHeader."No.");
        OrderData.Add('customerNo', SalesHeader."Sell-to Customer No.");
        OrderData.Add('orderDate', SalesHeader."Order Date");
        OrderData.Add('documentType', Format(SalesHeader."Document Type"));

        // Customer details (nested object)
        AddCustomerDetails(OrderData, SalesHeader."Sell-to Customer No.");

        // Order lines (array of objects)
        SalesLine.SetRange("Document Type", SalesHeader."Document Type");
        SalesLine.SetRange("Document No.", SalesHeader."No.");
        if SalesLine.FindSet() then
            repeat
                LinesArray.Add(BuildLinePayload(SalesLine));
            until SalesLine.Next() = 0;

        OrderData.Add('lines', LinesArray);

        // Totals summary
        AddOrderTotals(OrderData, SalesHeader);

        Payload.Add('order', OrderData);
        exit(Payload);
    end;

    local procedure BuildLinePayload(SalesLine: Record "Sales Line"): JsonObject
    var
        LineData: JsonObject;
    begin
        LineData.Add('lineNo', SalesLine."Line No.");
        LineData.Add('type', Format(SalesLine.Type));
        LineData.Add('no', SalesLine."No.");
        LineData.Add('description', SalesLine.Description);
        LineData.Add('quantity', SalesLine.Quantity);
        LineData.Add('unitPrice', SalesLine."Unit Price");
        LineData.Add('lineAmount', SalesLine."Line Amount");
        LineData.Add('discountPercent', SalesLine."Line Discount %");

        exit(LineData);
    end;
}
```

## Dynamic Payload Construction

```al
codeunit 50304 "Dynamic Payload Constructor"
{
    procedure BuildDynamicPayload(SourceTable: Record "Event Source Table"; PayloadTemplate: Text): JsonObject
    var
        Payload: JsonObject;
        TemplateObject: JsonObject;
        FieldMappings: JsonObject;
    begin
        TemplateObject.ReadFrom(PayloadTemplate);
        TemplateObject.Get('fieldMappings', FieldMappings);

        ProcessFieldMappings(Payload, SourceTable, FieldMappings);
        ApplyTransformations(Payload, TemplateObject);

        exit(Payload);
    end;

    local procedure ProcessFieldMappings(var Payload: JsonObject; SourceTable: Record "Event Source Table"; FieldMappings: JsonObject)
    var
        FieldMapping: JsonToken;
        FieldName: Text;
        SourceFieldName: Text;
        FieldValue: Text;
    begin
        foreach FieldName in FieldMappings.Keys() do begin
            FieldMappings.Get(FieldName, FieldMapping);
            SourceFieldName := FieldMapping.AsValue().AsText();

            FieldValue := GetFieldValue(SourceTable, SourceFieldName);
            Payload.Add(FieldName, FieldValue);
        end;
    end;

    local procedure GetFieldValue(SourceTable: Record "Event Source Table"; FieldName: Text): Text
    var
        RecordRef: RecordRef;
        FieldRef: FieldRef;
    begin
        RecordRef.GetTable(SourceTable);

        if RecordRef.FieldExist(FieldName) then begin
            FieldRef := RecordRef.Field(FieldName);
            exit(Format(FieldRef.Value));
        end;

        exit('');
    end;
}
```

## Payload Validation Pattern

```al
codeunit 50305 "Payload Validation Engine"
{
    procedure ValidatePayload(Payload: JsonObject; SchemaName: Code[50]): Boolean
    var
        ValidationRules: Record "Payload Validation Rule";
        ValidationResult: Boolean;
    begin
        ValidationResult := true;

        ValidationRules.SetRange("Schema Name", SchemaName);
        ValidationRules.SetRange(Active, true);

        if ValidationRules.FindSet() then
            repeat
                if not ValidateRule(Payload, ValidationRules) then
                    ValidationResult := false;
            until ValidationRules.Next() = 0;

        exit(ValidationResult);
    end;

    local procedure ValidateRule(Payload: JsonObject; ValidationRule: Record "Payload Validation Rule"): Boolean
    var
        FieldToken: JsonToken;
        FieldValue: Text;
    begin
        if not Payload.Get(ValidationRule."Field Path", FieldToken) then begin
            if ValidationRule.Required then
                exit(false);
            exit(true);
        end;

        FieldValue := FieldToken.AsValue().AsText();

        case ValidationRule."Validation Type" of
            ValidationRule."Validation Type"::"Data Type":
                exit(ValidateDataType(FieldValue, ValidationRule."Expected Value"));
            ValidationRule."Validation Type"::"String Length":
                exit(ValidateStringLength(FieldValue, ValidationRule."Min Length", ValidationRule."Max Length"));
            ValidationRule."Validation Type"::"Value Range":
                exit(ValidateValueRange(FieldValue, ValidationRule."Min Value", ValidationRule."Max Value"));
        end;

        exit(true);
    end;
}
```

This sample demonstrates comprehensive event payload design patterns including structured payloads, versioning, optimization, hierarchical data, dynamic construction, and validation strategies.