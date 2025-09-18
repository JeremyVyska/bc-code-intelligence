# Case Statement Syntax Structure Examples

## Basic Case Statement
```al
procedure ProcessDocumentType(DocumentType: Enum "Document Type")
begin
    case DocumentType of
        DocumentType::Quote:
            ProcessQuote();
        DocumentType::Order:
            ProcessOrder();
        DocumentType::"Credit Memo":
            ProcessCreditMemo();
        else
            Error('Unsupported document type: %1', DocumentType);
    end;
end;
```

## Case with Multiple Values
```al
procedure GetDocumentStatus(Status: Integer): Text
begin
    case Status of
        1, 2, 3:
            exit('Draft');
        4, 5:
            exit('Pending Approval');
        6:
            exit('Approved');
        else
            exit('Unknown');
    end;
end;
```

## Case with Ranges
```al
procedure CategorizeAmount(Amount: Decimal): Text
begin
    case true of
        (Amount >= 0) and (Amount <= 1000):
            exit('Small');
        (Amount > 1000) and (Amount <= 10000):
            exit('Medium');
        Amount > 10000:
            exit('Large');
        else
            exit('Invalid');
    end;
end;
```

## Complex Case Logic
```al
procedure HandleInventoryPosting(ItemType: Enum "Item Type"; LocationCode: Code[10])
begin
    case ItemType of
        ItemType::Inventory:
            begin
                if LocationCode <> '' then
                    PostToLocation(LocationCode)
                else
                    PostToDefaultLocation();
            end;
        ItemType::Service:
            PostServiceItem();
        ItemType::"Non-Inventory":
            LogNonInventoryTransaction();
    end;
end;
```