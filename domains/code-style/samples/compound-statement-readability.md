# AL Compound Statement Readability Examples

## Basic Conditional Enhancement

### Before: Single Statement
```al
if CustomerExists then
    CreateOrder();
```

### After: Compound Statement
```al
if CustomerExists then begin
    CreateOrder();
end;
```

## Complex Conditional Logic

### Nested Conditions with Clear Structure
```al
if ValidateCustomer(CustomerNo) then begin
    if CheckCreditLimit(CustomerNo, OrderAmount) then begin
        CreateSalesOrder(CustomerNo, OrderAmount);
        UpdateCustomerBalance(CustomerNo, OrderAmount);
        SendOrderConfirmation(CustomerNo);
    end else begin
        LogCreditLimitExceeded(CustomerNo, OrderAmount);
        NotifyCustomerService(CustomerNo);
    end;
end else begin
    LogInvalidCustomer(CustomerNo);
    RaiseCustomerValidationError(CustomerNo);
end;
```

## Loop Structure Clarification

### For Loop with Single Statement
```al
for i := 1 to ItemCount do begin
    ProcessItem(i);
end;
```

### While Loop with Multiple Operations
```al
while not ItemLedgerEntry.IsEmpty() do begin
    ItemLedgerEntry.CalcFields("Remaining Quantity");
    if ItemLedgerEntry."Remaining Quantity" > 0 then begin
        UpdateInventoryPosition(ItemLedgerEntry);
        LogInventoryTransaction(ItemLedgerEntry);
    end;
    ItemLedgerEntry.Next();
end;
```

## Error Handling Organization

### Exception Handling with Compound Blocks
```al
if not ValidateOrderData(SalesHeader) then begin
    LogValidationError(SalesHeader."No.");
    NotifyUser('Order validation failed');
    exit(false);
end;

try
    ProcessSalesOrder(SalesHeader);
except
    on E: Exception do begin
        LogException(E.Message, SalesHeader."No.");
        RollbackTransaction();
        RaiseOrderProcessingError(E.Message);
    end;
end;
```

## Case Statement Structure

### Clear Case Branch Organization
```al
case DocumentType of
    DocumentType::Quote:
        begin
            ValidateQuoteData();
            UpdateQuoteStatus();
        end;
    DocumentType::Order:
        begin
            ValidateOrderData();
            ReserveInventory();
            UpdateOrderStatus();
        end;
    DocumentType::Invoice:
        begin
            ValidateInvoiceData();
            PostToGeneralLedger();
            UpdateInvoiceStatus();
        end;
    else
        begin
            LogUnsupportedDocumentType(DocumentType);
            Error('Unsupported document type: %1', DocumentType);
        end;
end;
```