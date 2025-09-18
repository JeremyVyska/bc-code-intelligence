# AL Nested Compound Statement Best Practices Examples

## Manageable Nesting Depth Examples

### Good: Three-Level Nesting with Clear Purpose
```al
local procedure ProcessCustomerOrder(var SalesHeader: Record "Sales Header")
begin
    if ValidateCustomerData(SalesHeader) then begin
        if CheckInventoryAvailability(SalesHeader) then begin
            if ValidateCreditLimit(SalesHeader) then begin
                ProcessApprovedOrder(SalesHeader);
            end else begin
                HandleCreditLimitExceeded(SalesHeader);
            end;
        end else begin
            HandleInventoryShortage(SalesHeader);
        end;
    end else begin
        HandleCustomerValidationFailure(SalesHeader);
    end;
end;
```

### Better: Extracted Logic for Clarity
```al
local procedure ProcessCustomerOrder(var SalesHeader: Record "Sales Header")
begin
    if not ValidateCustomerData(SalesHeader) then begin
        HandleCustomerValidationFailure(SalesHeader);
        exit;
    end;
    
    if not CheckInventoryAvailability(SalesHeader) then begin
        HandleInventoryShortage(SalesHeader);
        exit;
    end;
    
    ProcessOrderWithCreditCheck(SalesHeader);
end;

local procedure ProcessOrderWithCreditCheck(var SalesHeader: Record "Sales Header")
begin
    if ValidateCreditLimit(SalesHeader) then begin
        ProcessApprovedOrder(SalesHeader);
    end else begin
        HandleCreditLimitExceeded(SalesHeader);
    end;
end;
```

## Validation Cascade Patterns

### Structured Validation with Clear Flow
```al
local procedure ValidateOrderData(var SalesHeader: Record "Sales Header"): Boolean
var
    Customer: Record Customer;
    Item: Record Item;
begin
    if SalesHeader."Sell-to Customer No." = '' then begin
        LogValidationError('Customer number is required');
        exit(false);
    end;
    
    if Customer.Get(SalesHeader."Sell-to Customer No.") then begin
        if Customer.Blocked = Customer.Blocked::All then begin
            LogValidationError('Customer is blocked for all transactions');
            exit(false);
        end;
        
        if ValidateOrderLines(SalesHeader) then begin
            if ValidateOrderAmount(SalesHeader, Customer) then begin
                exit(true);
            end else begin
                LogValidationError('Order amount validation failed');
                exit(false);
            end;
        end else begin
            LogValidationError('Order line validation failed');
            exit(false);
        end;
    end else begin
        LogValidationError('Customer not found');
        exit(false);
    end;
end;
```

## Error Handling Hierarchies

### Layered Exception Handling
```al
local procedure ProcessDocumentBatch(var DocumentBuffer: Record "Document Buffer")
var
    ProcessedCount: Integer;
    ErrorCount: Integer;
begin
    if DocumentBuffer.FindSet() then begin
        repeat
            try
                if ValidateDocumentStructure(DocumentBuffer) then begin
                    try
                        ProcessSingleDocument(DocumentBuffer);
                        ProcessedCount += 1;
                    except
                        on E: Exception do begin
                            LogDocumentProcessingError(DocumentBuffer."Document No.", E.Message);
                            ErrorCount += 1;
                        end;
                    end;
                end else begin
                    LogDocumentValidationError(DocumentBuffer."Document No.");
                    ErrorCount += 1;
                end;
            except
                on E: Exception do begin
                    LogCriticalError('Batch processing failed', E.Message);
                    Error('Batch processing terminated due to critical error');
                end;
            end;
        until DocumentBuffer.Next() = 0;
        
        LogBatchSummary(ProcessedCount, ErrorCount);
    end;
end;
```