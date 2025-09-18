# Case Statement Performance Optimization - AL Code Samples

## Frequency-Based Ordering

```al
procedure ProcessTransactionOptimized(TransactionType: Text[20]; Amount: Decimal): Boolean
begin
    // Order by frequency - most common first
    case TransactionType of
        'SALE':        // 60% of transactions
            exit(ProcessSaleTransaction(Amount));
        'PAYMENT':     // 25% of transactions
            exit(ProcessPaymentTransaction(Amount));
        'REFUND':      // 10% of transactions
            exit(ProcessRefundTransaction(Amount));
        else
            exit(ProcessUnknownTransaction(TransactionType, Amount));
    end;
end;
```

## Enum vs Text Performance

```al
// Faster enum-based approach
procedure ProcessTransactionEnum(TransactionType: Enum "Transaction Type"; Amount: Decimal): Boolean
begin
    case TransactionType of
        TransactionType::Sale:
            exit(ProcessSaleTransaction(Amount));
        TransactionType::Payment:
            exit(ProcessPaymentTransaction(Amount));
        else
            exit(false);
    end;
end;
```

## Dictionary Alternative for Large Case Sets

```al
procedure ProcessWithDictionary(ProcessingType: Text[20]; Data: Text): Text
var
    ProcessingFunctions: Dictionary of [Text, Integer];
    FunctionId: Integer;
begin
    ProcessingFunctions.Add('FORMAT', 1);
    ProcessingFunctions.Add('VALIDATE', 2);
    ProcessingFunctions.Add('TRANSFORM', 3);
    
    if ProcessingFunctions.Get(ProcessingType, FunctionId) then
        case FunctionId of
            1: exit(FormatData(Data));
            2: exit(ValidateData(Data));
            3: exit(TransformData(Data));
        end;
    
    exit('Unknown processing type');
end;
```