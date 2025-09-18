# AL Case Action Formatting - Code Examples

## Good Examples

### Single Statement Actions
```al
procedure ProcessOrderStatus(Status: Option Open,Released,Pending)
begin
    case Status of
        Status::Open:
            Message('Order is open for editing');
        Status::Released:
            Message('Order has been released');
        Status::Pending:
            Message('Order is pending approval');
    end;
end;
```

### Multi-Statement Actions with Begin-End
```al
procedure HandleDocumentType(DocumentType: Option Quote,Order,Invoice)
var
    SalesHeader: Record "Sales Header";
begin
    case DocumentType of
        DocumentType::Quote:
            begin
                SalesHeader."Document Type" := SalesHeader."Document Type"::Quote;
                SalesHeader."Quote Valid Until Date" := CalcDate('+30D', Today);
                SalesHeader.Insert();
            end;
        DocumentType::Order:
            begin
                SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
                SalesHeader."Shipment Date" := CalcDate('+7D', Today);
                SalesHeader.Insert();
                SendOrderConfirmation(SalesHeader);
            end;
        DocumentType::Invoice:
            begin
                SalesHeader."Document Type" := SalesHeader."Document Type"::Invoice;
                SalesHeader."Due Date" := CalcDate('+30D', Today);
                SalesHeader.Insert();
                PostInvoice(SalesHeader);
            end;
    end;
end;
```

### Complex Case with Nested Logic
```al
procedure ProcessPaymentMethod(PaymentMethod: Option Cash,CreditCard,BankTransfer,Check)
var
    PaymentEntry: Record "Payment Entry";
    BankAccount: Record "Bank Account";
begin
    case PaymentMethod of
        PaymentMethod::Cash:
            begin
                PaymentEntry."Payment Type" := PaymentEntry."Payment Type"::Cash;
                PaymentEntry."Processing Fee" := 0;
                PaymentEntry.Insert();
            end;
        PaymentMethod::CreditCard:
            begin
                PaymentEntry."Payment Type" := PaymentEntry."Payment Type"::"Credit Card";
                PaymentEntry."Processing Fee" := PaymentEntry.Amount * 0.03;
                if PaymentEntry."Processing Fee" < 2.50 then
                    PaymentEntry."Processing Fee" := 2.50;
                PaymentEntry.Insert();
            end;
        PaymentMethod::BankTransfer:
            begin
                if BankAccount.Get(PaymentEntry."Bank Account No.") then begin
                    PaymentEntry."Payment Type" := PaymentEntry."Payment Type"::"Bank Transfer";
                    PaymentEntry."Processing Fee" := 5.00;
                    PaymentEntry.Insert();
                    InitiateBankTransfer(PaymentEntry);
                end else
                    Error('Bank account not found');
            end;
        PaymentMethod::Check:
            begin
                PaymentEntry."Payment Type" := PaymentEntry."Payment Type"::Check;
                PaymentEntry."Processing Fee" := 1.00;
                PaymentEntry."Hold Period" := 3;
                PaymentEntry.Insert();
            end;
    end;
end;
```

## Bad Examples

### Inconsistent Formatting
```al
procedure BadCaseFormatting(Status: Option Open,Released,Pending)
begin
    case Status of
        Status::Open: Message('Order is open');
        Status::Released:
        begin
            Message('Order released');
            UpdateStatus();
        end;
        Status::Pending: begin Message('Pending approval'); RequestApproval(); end;
    end;
end;
```

### Missing Begin-End for Multi-Statement Actions
```al
procedure MissingBeginEnd(DocumentType: Option Quote,Order,Invoice)
var
    SalesHeader: Record "Sales Header";
begin
    case DocumentType of
        DocumentType::Quote:
            SalesHeader."Document Type" := SalesHeader."Document Type"::Quote;
            SalesHeader."Quote Valid Until Date" := CalcDate('+30D', Today);  // Wrong: not in begin-end block
        DocumentType::Order:
            SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
            SalesHeader."Shipment Date" := CalcDate('+7D', Today);  // Wrong: not in begin-end block
    end;
end;
```

### Poor Indentation
```al
procedure PoorIndentation(Status: Option Open,Released,Pending)
begin
case Status of
Status::Open:
Message('Order is open');
Status::Released:
begin
Message('Order released');
UpdateStatus();
end;
Status::Pending:
begin
Message('Pending approval');
RequestApproval();
end;
end;
end;
```

### Unnecessary Begin-End for Single Statements
```al
procedure UnnecessaryBeginEnd(Status: Option Open,Released,Pending)
begin
    case Status of
        Status::Open:
            begin  // Unnecessary for single statement
                Message('Order is open');
            end;
        Status::Released:
            begin  // Unnecessary for single statement
                Message('Order released');
            end;
        Status::Pending:
            begin  // Unnecessary for single statement
                Message('Pending approval');
            end;
    end;
end;
```

## Best Practices

1. **Single statements** don't need begin-end blocks
2. **Multiple statements** must use begin-end blocks
3. **Consistent indentation** for all case branches
4. **Align case labels** for visual clarity
5. **Group related case logic** with appropriate spacing