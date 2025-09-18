# Unnecessary Else Elimination - AL Code Examples

## Good Examples

### Early Return Pattern
```al
procedure ValidateCustomerCredit(CustomerNo: Code[20]): Boolean
var
    Customer: Record Customer;
    CustLedgerEntry: Record "Cust. Ledger Entry";
    TotalOutstanding: Decimal;
begin
    // Early exit for invalid customer
    if not Customer.Get(CustomerNo) then
        exit(false);

    // Early exit if credit check disabled
    if not Customer."Credit Limit (LCY)" > 0 then
        exit(true);

    // Calculate outstanding amount
    CustLedgerEntry.SetRange("Customer No.", CustomerNo);
    CustLedgerEntry.SetRange(Open, true);
    CustLedgerEntry.CalcSums("Remaining Amt. (LCY)");
    TotalOutstanding := CustLedgerEntry."Remaining Amt. (LCY)";

    // Direct return without else
    exit(TotalOutstanding <= Customer."Credit Limit (LCY)");
end;
```

### Guard Clause Implementation
```al
procedure ProcessSalesOrder(DocumentNo: Code[20]): Boolean
var
    SalesHeader: Record "Sales Header";
    SalesLine: Record "Sales Line";
begin
    // Guard clause - early exit
    if DocumentNo = '' then
        exit(false);

    // Guard clause - record validation
    if not SalesHeader.Get(SalesHeader."Document Type"::Order, DocumentNo) then
        exit(false);

    // Guard clause - status validation
    if SalesHeader.Status <> SalesHeader.Status::Open then
        exit(false);

    // Main processing logic
    SalesLine.SetRange("Document Type", SalesHeader."Document Type");
    SalesLine.SetRange("Document No.", SalesHeader."No.");
    if SalesLine.FindSet() then
        repeat
            ProcessSalesLine(SalesLine);
        until SalesLine.Next() = 0;

    exit(true);
end;
```

### Streamlined Control Flow
```al
procedure CalculateItemCost(ItemNo: Code[20]; Quantity: Decimal): Decimal
var
    Item: Record Item;
    ItemLedgerEntry: Record "Item Ledger Entry";
begin
    // Early validation without nested else
    if ItemNo = '' then
        exit(0);

    if Quantity <= 0 then
        exit(0);

    if not Item.Get(ItemNo) then
        exit(0);

    // Calculate based on costing method
    case Item."Costing Method" of
        Item."Costing Method"::FIFO:
            exit(CalculateFIFOCost(ItemNo, Quantity));
        Item."Costing Method"::LIFO:
            exit(CalculateLIFOCost(ItemNo, Quantity));
        Item."Costing Method"::Average:
            exit(Item."Unit Cost" * Quantity);
        else
            exit(Item."Last Direct Cost" * Quantity);
    end;
end;
```

## Bad Examples

### Unnecessary Else Chains
```al
procedure ValidateCustomerCredit(CustomerNo: Code[20]): Boolean
var
    Customer: Record Customer;
    CustLedgerEntry: Record "Cust. Ledger Entry";
    TotalOutstanding: Decimal;
begin
    if CustomerNo = '' then begin
        exit(false);
    end else begin
        if Customer.Get(CustomerNo) then begin
            if Customer."Credit Limit (LCY)" > 0 then begin
                CustLedgerEntry.SetRange("Customer No.", CustomerNo);
                CustLedgerEntry.SetRange(Open, true);
                CustLedgerEntry.CalcSums("Remaining Amt. (LCY)");
                TotalOutstanding := CustLedgerEntry."Remaining Amt. (LCY)";
                if TotalOutstanding <= Customer."Credit Limit (LCY)" then begin
                    exit(true);
                end else begin
                    exit(false);
                end;
            end else begin
                exit(true);
            end;
        end else begin
            exit(false);
        end;
    end;
end;
```

### Nested Else Complexity
```al
procedure ProcessDocument(DocumentType: Integer; DocumentNo: Code[20]): Boolean
var
    ProcessingResult: Boolean;
begin
    if DocumentType = 1 then begin
        ProcessingResult := ProcessSalesDocument(DocumentNo);
        if ProcessingResult then begin
            exit(true);
        end else begin
            exit(false);
        end;
    end else begin
        if DocumentType = 2 then begin
            ProcessingResult := ProcessPurchaseDocument(DocumentNo);
            if ProcessingResult then begin
                exit(true);
            end else begin
                exit(false);
            end;
        end else begin
            if DocumentType = 3 then begin
                ProcessingResult := ProcessServiceDocument(DocumentNo);
                if ProcessingResult then begin
                    exit(true);
                end else begin
                    exit(false);
                end;
            end else begin
                exit(false);
            end;
        end;
    end;
end;
```

## Best Practices

### Boolean Return Simplification
```al
procedure IsItemBlocked(ItemNo: Code[20]): Boolean
var
    Item: Record Item;
begin
    // Good: Direct boolean return
    if not Item.Get(ItemNo) then
        exit(true);

    exit(Item.Blocked);

    // Avoid: Unnecessary if-then-else for boolean
    // if Item.Blocked then
    //     exit(true)
    // else
    //     exit(false);
end;
```

### Case Statement Optimization
```al
procedure GetDocumentStatus(Status: Option Open,Released,Pending): Text
begin
    // Good: Case with early returns
    case Status of
        Status::Open:
            exit('OPEN');
        Status::Released:
            exit('RELEASED');
        Status::Pending:
            exit('PENDING');
        else
            exit('UNKNOWN');
    end;

    // Avoid: Nested if-else chains
    // if Status = Status::Open then
    //     exit('OPEN')
    // else if Status = Status::Released then
    //     exit('RELEASED')
    // else if Status = Status::Pending then
    //     exit('PENDING')
    // else
    //     exit('UNKNOWN');
end;
```

### Exception Handling Pattern
```al
procedure SafelyProcessRecord(RecordID: RecordID): Boolean
var
    RecordRef: RecordRef;
begin
    // Early exit on invalid input
    if RecordID.TableNo = 0 then
        exit(false);

    // Early exit if record doesn't exist
    if not RecordRef.Get(RecordID) then
        exit(false);

    // Early exit if record is already processed
    if IsRecordProcessed(RecordRef) then
        exit(true);

    // Main processing
    exit(ProcessRecord(RecordRef));
end;
```

## Implementation Guidelines

1. **Replace nested if-else with early returns** - Reduce cognitive complexity
2. **Use guard clauses for validation** - Handle edge cases at procedure start
3. **Simplify boolean returns** - Return boolean expressions directly
4. **Prefer case statements** - For multiple discrete value comparisons
5. **Apply consistent patterns** - Use same structure across similar procedures
6. **Test thoroughly** - Ensure refactored logic maintains original behavior