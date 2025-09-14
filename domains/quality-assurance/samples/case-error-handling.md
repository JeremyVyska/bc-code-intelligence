# Case Statement Error Handling - AL Code Samples

## Basic Error Handling Pattern

```al
procedure ProcessOrderStatus(OrderStatus: Enum "Sales Document Status")
begin
    case OrderStatus of
        OrderStatus::Open:
            ProcessOpenOrder();
        OrderStatus::Released:
            ProcessReleasedOrder();
        OrderStatus::"Pending Approval":
            ProcessPendingOrder();
        else
            Error('Unsupported order status: %1', OrderStatus);
    end;
end;
```

## Safe Processing with TryFunction

```al
[TryFunction]
procedure ProcessOrderStatusSafe(OrderStatus: Enum "Sales Document Status"): Boolean
begin
    case OrderStatus of
        OrderStatus::Open:
            exit(TryProcessOpenOrder());
        OrderStatus::Released:
            exit(TryProcessReleasedOrder());
        else
            exit(false);
    end;
end;
```

## Exception Recovery Pattern

```al
procedure ProcessWithRecovery(ProcessingType: Text; MaxAttempts: Integer)
var
    Attempts: Integer;
    Success: Boolean;
begin
    repeat
        Attempts += 1;
        case ProcessingType of
            'STANDARD':
                Success := TryStandardProcessing();
            'FALLBACK':
                Success := TryFallbackProcessing();
            else
                Success := false;
        end;
        
        if not Success and (Attempts < MaxAttempts) then
            Sleep(1000);
    until Success or (Attempts >= MaxAttempts);
    
    if not Success then
        Error('Failed after %1 attempts', MaxAttempts);
end;
```