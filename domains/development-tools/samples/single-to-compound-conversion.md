# AL Single to Compound Statement Conversion Examples

## Basic Conditional Conversion

### Before: Single Statement If
```al
if CustomerNo <> '' then
    ValidateCustomer(CustomerNo);
```

### After: Compound Statement
```al
if CustomerNo <> '' then begin
    ValidateCustomer(CustomerNo);
end;
```

## Complex Conditional Transformation

### Before: Mixed Single/Compound Pattern
```al
if ValidateOrder(SalesHeader) then
    ProcessOrder(SalesHeader)
else begin
    LogOrderError(SalesHeader."No.");
    NotifyUser('Order validation failed');
end;
```

### After: Consistent Compound Pattern
```al
if ValidateOrder(SalesHeader) then begin
    ProcessOrder(SalesHeader);
end else begin
    LogOrderError(SalesHeader."No.");
    NotifyUser('Order validation failed');
end;
```

## Loop Statement Conversion

### Before: Single Statement For Loop
```al
for i := 1 to RecordCount do
    ProcessRecord(i);
```

### After: Compound For Loop
```al
for i := 1 to RecordCount do begin
    ProcessRecord(i);
end;
```

## Case Statement Enhancement

### Before: Mixed Case Pattern
```al
case DocumentType of
    DocumentType::Quote:
        ProcessQuote(DocumentNo);
    DocumentType::Order:
        begin
            ValidateOrder(DocumentNo);
            ProcessOrder(DocumentNo);
        end;
    DocumentType::Invoice:
        ProcessInvoice(DocumentNo);
end;
```

### After: Consistent Case Pattern
```al
case DocumentType of
    DocumentType::Quote:
        begin
            ProcessQuote(DocumentNo);
        end;
    DocumentType::Order:
        begin
            ValidateOrder(DocumentNo);
            ProcessOrder(DocumentNo);
        end;
    DocumentType::Invoice:
        begin
            ProcessInvoice(DocumentNo);
        end;
end;
```