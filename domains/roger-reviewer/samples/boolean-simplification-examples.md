# Boolean Expression Simplification in AL - Code Examples

## Overview
Complex boolean expressions in AL code can significantly impact readability, maintainability, and performance. This sample demonstrates systematic approaches to simplifying boolean logic while maintaining functionality and improving code clarity.

## Fundamental Simplification Patterns

### Example 1: Basic De Morgan's Laws Application

**Problematic Code:**
```al
procedure ValidateCustomerOrder(SalesHeader: Record "Sales Header"): Boolean
var
    Customer: Record Customer;
begin
    Customer.Get(SalesHeader."Sell-to Customer No.");

    // Complex negative condition - hard to read and understand
    if not (Customer.Blocked = Customer.Blocked::" ") or
       not (SalesHeader.Status = SalesHeader.Status::Open) or
       not (SalesHeader."Document Date" >= Today()) then
        exit(false);

    exit(true);
end;
```

**Simplified Code:**
```al
procedure ValidateCustomerOrder(SalesHeader: Record "Sales Header"): Boolean
var
    Customer: Record Customer;
    CustomerBlocked: Boolean;
    OrderNotOpen: Boolean;
    OrderDateInvalid: Boolean;
begin
    Customer.Get(SalesHeader."Sell-to Customer No.");

    // Apply De Morgan's law: not (A or B or C) = (not A) and (not B) and (not C)
    // Then simplify by removing double negatives
    CustomerBlocked := Customer.Blocked <> Customer.Blocked::" ";
    OrderNotOpen := SalesHeader.Status <> SalesHeader.Status::Open;
    OrderDateInvalid := SalesHeader."Document Date" < Today();

    // Clear positive logic: ALL conditions must be false for validation to pass
    if CustomerBlocked or OrderNotOpen or OrderDateInvalid then
        exit(false);

    exit(true);
end;
```

### Example 2: Condition Extraction and Grouping

**Problematic Code:**
```al
procedure CanPostSalesDocument(SalesHeader: Record "Sales Header"): Boolean
var
    SalesLine: Record "Sales Line";
    GeneralLedgerSetup: Record "General Ledger Setup";
    UserSetup: Record "User Setup";
begin
    GeneralLedgerSetup.Get();
    UserSetup.Get(UserId());

    // Extremely complex boolean expression mixing different concerns
    if (SalesHeader.Status = SalesHeader.Status::Released) and
       (SalesHeader."Posting Date" <> 0D) and
       (SalesHeader."Document Date" <> 0D) and
       ((SalesHeader."Posting Date" >= GeneralLedgerSetup."Allow Posting From") and
        (SalesHeader."Posting Date" <= GeneralLedgerSetup."Allow Posting To")) and
       ((UserSetup."Allow Posting From" = 0D) or (SalesHeader."Posting Date" >= UserSetup."Allow Posting From")) and
       ((UserSetup."Allow Posting To" = 0D) or (SalesHeader."Posting Date" <= UserSetup."Allow Posting To")) and
       (SalesHeader."Sell-to Customer No." <> '') and
       (SalesHeader.Amount > 0) then
        exit(true)
    else
        exit(false);
end;
```

**Simplified Code:**
```al
procedure CanPostSalesDocument(SalesHeader: Record "Sales Header"): Boolean
begin
    // Break complex validation into logical groups
    if not IsDocumentStatusValid(SalesHeader) then
        exit(false);

    if not AreDatesValid(SalesHeader) then
        exit(false);

    if not IsPostingDateAllowed(SalesHeader."Posting Date") then
        exit(false);

    if not IsDocumentContentValid(SalesHeader) then
        exit(false);

    exit(true);
end;

local procedure IsDocumentStatusValid(SalesHeader: Record "Sales Header"): Boolean
begin
    exit(SalesHeader.Status = SalesHeader.Status::Released);
end;

local procedure AreDatesValid(SalesHeader: Record "Sales Header"): Boolean
begin
    exit((SalesHeader."Posting Date" <> 0D) and (SalesHeader."Document Date" <> 0D));
end;

local procedure IsPostingDateAllowed(PostingDate: Date): Boolean
var
    GeneralLedgerSetup: Record "General Ledger Setup";
    UserSetup: Record "User Setup";
begin
    GeneralLedgerSetup.Get();

    // Check system-wide posting date restrictions
    if not ((PostingDate >= GeneralLedgerSetup."Allow Posting From") and
            (PostingDate <= GeneralLedgerSetup."Allow Posting To")) then
        exit(false);

    // Check user-specific posting date restrictions
    if UserSetup.Get(UserId()) then
        exit(IsWithinUserDateRange(PostingDate, UserSetup));

    exit(true);
end;

local procedure IsWithinUserDateRange(PostingDate: Date; UserSetup: Record "User Setup"): Boolean
var
    WithinFromDate: Boolean;
    WithinToDate: Boolean;
begin
    WithinFromDate := (UserSetup."Allow Posting From" = 0D) or (PostingDate >= UserSetup."Allow Posting From");
    WithinToDate := (UserSetup."Allow Posting To" = 0D) or (PostingDate <= UserSetup."Allow Posting To");

    exit(WithinFromDate and WithinToDate);
end;

local procedure IsDocumentContentValid(SalesHeader: Record "Sales Header"): Boolean
begin
    exit((SalesHeader."Sell-to Customer No." <> '') and (SalesHeader.Amount > 0));
end;
```

## Advanced Boolean Simplification Techniques

### Example 3: Short-Circuit Evaluation Optimization

**Problematic Code:**
```al
procedure ValidateItemForSale(ItemNo: Code[20]; LocationCode: Code[10]; Quantity: Decimal): Boolean
var
    Item: Record Item;
    ItemLedgerEntry: Record "Item Ledger Entry";
    Location: Record Location;
    StockkeepingUnit: Record "Stockkeeping Unit";
    AvailableInventory: Decimal;
    IsReserved: Boolean;
    HasSpecialPricing: Boolean;
begin
    // Poor ordering - expensive operations performed regardless of simple checks
    Item.Get(ItemNo);
    Item.CalcFields(Inventory);
    AvailableInventory := CalculateAvailableInventory(ItemNo, LocationCode);
    IsReserved := CheckReservationStatus(ItemNo, LocationCode);
    HasSpecialPricing := CheckSpecialPricing(ItemNo);
    Location.Get(LocationCode);

    if (Item.Blocked = false) and
       (Item.Type = Item.Type::Inventory) and
       (Location."Use As In-Transit" = false) and
       (Quantity > 0) and
       (AvailableInventory >= Quantity) and
       (IsReserved = false) and
       (HasSpecialPricing = true) then
        exit(true)
    else
        exit(false);
end;
```

**Optimized Code:**
```al
procedure ValidateItemForSale(ItemNo: Code[20]; LocationCode: Code[10]; Quantity: Decimal): Boolean
var
    Item: Record Item;
    Location: Record Location;
begin
    // Fast validation first - fail early on simple checks
    if Quantity <= 0 then
        exit(false);

    if not Item.Get(ItemNo) then
        exit(false);

    // Basic item validation
    if not IsItemAvailableForSale(Item) then
        exit(false);

    // Location validation
    if not IsLocationValidForSale(LocationCode) then
        exit(false);

    // Expensive operations only after basic validation passes
    if not HasSufficientInventory(ItemNo, LocationCode, Quantity) then
        exit(false);

    if not IsInventoryUnreserved(ItemNo, LocationCode) then
        exit(false);

    if not HasValidPricing(ItemNo) then
        exit(false);

    exit(true);
end;

local procedure IsItemAvailableForSale(Item: Record Item): Boolean
begin
    exit((Item.Blocked = false) and (Item.Type = Item.Type::Inventory));
end;

local procedure IsLocationValidForSale(LocationCode: Code[10]): Boolean
var
    Location: Record Location;
begin
    if not Location.Get(LocationCode) then
        exit(false);

    exit(Location."Use As In-Transit" = false);
end;

local procedure HasSufficientInventory(ItemNo: Code[20]; LocationCode: Code[10]; RequiredQty: Decimal): Boolean
var
    AvailableInventory: Decimal;
begin
    AvailableInventory := CalculateAvailableInventory(ItemNo, LocationCode);
    exit(AvailableInventory >= RequiredQty);
end;

local procedure IsInventoryUnreserved(ItemNo: Code[20]; LocationCode: Code[10]): Boolean
begin
    exit(not CheckReservationStatus(ItemNo, LocationCode));
end;

local procedure HasValidPricing(ItemNo: Code[20]): Boolean
begin
    exit(CheckSpecialPricing(ItemNo));
end;
```

### Example 4: Complex Range and Set Validation

**Problematic Code:**
```al
procedure ValidateGLAccountPosting(GLAccountNo: Code[20]; PostingDate: Date; Amount: Decimal): Boolean
var
    GLAccount: Record "G/L Account";
    AccountingPeriod: Record "Accounting Period";
    GLSetup: Record "General Ledger Setup";
    UserSetup: Record "User Setup";
begin
    GLAccount.Get(GLAccountNo);
    GLSetup.Get();

    // Overly complex condition mixing different validation concerns
    if (GLAccount.Blocked = false) and
       (GLAccount."Direct Posting" = true) and
       ((GLAccount."Account Type" = GLAccount."Account Type"::Posting) or
        (GLAccount."Account Type" = GLAccount."Account Type"::"Begin-Total") or
        (GLAccount."Account Type" = GLAccount."Account Type"::"End-Total")) and
       (PostingDate <> 0D) and
       (PostingDate >= GLSetup."Allow Posting From") and
       (PostingDate <= GLSetup."Allow Posting To") and
       ((Amount > 0) or (Amount < 0)) and
       (Amount <> 0) and
       ((UserSetup.Get(UserId()) and
         ((UserSetup."Allow Posting From" = 0D) or (PostingDate >= UserSetup."Allow Posting From")) and
         ((UserSetup."Allow Posting To" = 0D) or (PostingDate <= UserSetup."Allow Posting To"))) or
        (not UserSetup.Get(UserId()))) then
        exit(true)
    else
        exit(false);
end;
```

**Simplified Code:**
```al
procedure ValidateGLAccountPosting(GLAccountNo: Code[20]; PostingDate: Date; Amount: Decimal): Boolean
var
    GLAccount: Record "G/L Account";
begin
    if not GLAccount.Get(GLAccountNo) then
        exit(false);

    // Separate validation concerns for clarity
    exit(IsAccountValidForPosting(GLAccount) and
         IsPostingDateValid(PostingDate) and
         IsAmountValid(Amount));
end;

local procedure IsAccountValidForPosting(GLAccount: Record "G/L Account"): Boolean
begin
    if GLAccount.Blocked then
        exit(false);

    if not GLAccount."Direct Posting" then
        exit(false);

    // Simplified account type check using set inclusion
    exit(GLAccount."Account Type" in [
        GLAccount."Account Type"::Posting,
        GLAccount."Account Type"::"Begin-Total",
        GLAccount."Account Type"::"End-Total"
    ]);
end;

local procedure IsPostingDateValid(PostingDate: Date): Boolean
begin
    if PostingDate = 0D then
        exit(false);

    exit(IsWithinSystemDateRange(PostingDate) and IsWithinUserDateRange(PostingDate));
end;

local procedure IsWithinSystemDateRange(PostingDate: Date): Boolean
var
    GLSetup: Record "General Ledger Setup";
begin
    GLSetup.Get();
    exit((PostingDate >= GLSetup."Allow Posting From") and
         (PostingDate <= GLSetup."Allow Posting To"));
end;

local procedure IsWithinUserDateRange(PostingDate: Date): Boolean
var
    UserSetup: Record "User Setup";
begin
    // If no user setup exists, allow posting (system rules apply)
    if not UserSetup.Get(UserId()) then
        exit(true);

    // Check user-specific date restrictions
    if (UserSetup."Allow Posting From" <> 0D) and (PostingDate < UserSetup."Allow Posting From") then
        exit(false);

    if (UserSetup."Allow Posting To" <> 0D) and (PostingDate > UserSetup."Allow Posting To") then
        exit(false);

    exit(true);
end;

local procedure IsAmountValid(Amount: Decimal): Boolean
begin
    // Simplified: amount must be non-zero (positive or negative)
    exit(Amount <> 0);
end;
```

## Performance-Oriented Boolean Optimization

### Example 5: Database Query Optimization Through Boolean Logic

**Problematic Code:**
```al
procedure FindEligibleCustomersForPromotion(): List of [Code[20]]
var
    Customer: Record Customer;
    CustomerList: List of [Code[20]];
    SalesHeader: Record "Sales Header";
    TotalSales: Decimal;
    HasRecentOrder: Boolean;
    IsHighValue: Boolean;
    IsActiveAccount: Boolean;
begin
    // Inefficient: processes all customers regardless of basic criteria
    if Customer.FindSet() then
        repeat
            // Multiple database hits per customer
            Customer.CalcFields("Sales (LCY)");
            TotalSales := Customer."Sales (LCY)";

            SalesHeader.SetRange("Sell-to Customer No.", Customer."No.");
            SalesHeader.SetFilter("Order Date", '>=%1', CalcDate('-6M', Today()));
            HasRecentOrder := not SalesHeader.IsEmpty();

            IsHighValue := TotalSales > 50000;
            IsActiveAccount := Customer.Blocked = Customer.Blocked::" ";

            // Complex condition evaluated for every customer
            if (IsActiveAccount = true) and
               (IsHighValue = true) and
               (HasRecentOrder = true) and
               (Customer."Country/Region Code" = 'US') and
               (Customer."Customer Posting Group" <> 'CASH') then
                CustomerList.Add(Customer."No.");
        until Customer.Next() = 0;

    exit(CustomerList);
end;
```

**Optimized Code:**
```al
procedure FindEligibleCustomersForPromotion(): List of [Code[20]]
var
    Customer: Record Customer;
    CustomerList: List of [Code[20]];
begin
    // Pre-filter customers to reduce dataset
    SetCustomerFiltersForPromotion(Customer);

    if Customer.FindSet() then
        repeat
            if IsCustomerEligibleForPromotion(Customer) then
                CustomerList.Add(Customer."No.");
        until Customer.Next() = 0;

    exit(CustomerList);
end;

local procedure SetCustomerFiltersForPromotion(var Customer: Record Customer)
begin
    // Apply filters early to reduce record set
    Customer.SetRange(Blocked, Customer.Blocked::" ");  // Active accounts only
    Customer.SetRange("Country/Region Code", 'US');     // US customers only
    Customer.SetFilter("Customer Posting Group", '<>%1', 'CASH'); // Exclude cash customers
end;

local procedure IsCustomerEligibleForPromotion(Customer: Record Customer): Boolean
begin
    // Short-circuit evaluation: check expensive operations last
    exit(IsHighValueCustomer(Customer) and HasRecentActivity(Customer));
end;

local procedure IsHighValueCustomer(Customer: Record Customer): Boolean
begin
    Customer.CalcFields("Sales (LCY)");
    exit(Customer."Sales (LCY)" > 50000);
end;

local procedure HasRecentActivity(Customer: Record Customer): Boolean
var
    SalesHeader: Record "Sales Header";
begin
    SalesHeader.SetRange("Sell-to Customer No.", Customer."No.");
    SalesHeader.SetFilter("Order Date", '>=%1', CalcDate('-6M', Today()));
    exit(not SalesHeader.IsEmpty());
end;
```

## Best Practices for Boolean Expression Simplification

### Readability Improvements
1. **Extract conditions into well-named boolean variables** for complex expressions
2. **Use early return patterns** to avoid deeply nested if-else structures
3. **Group related conditions** into separate validation methods
4. **Apply De Morgan's laws** to eliminate complex negative conditions

### Performance Optimizations
1. **Order conditions by evaluation cost** - simple checks first, expensive operations last
2. **Use short-circuit evaluation** to avoid unnecessary calculations
3. **Pre-filter datasets** before applying complex boolean logic
4. **Cache expensive boolean calculations** when used multiple times

### Maintainability Patterns
1. **Separate concerns** - group related validation logic together
2. **Use meaningful method names** that clearly indicate validation purpose
3. **Avoid magic numbers and values** in boolean expressions
4. **Document complex business logic** with comments explaining the reasoning

### Common Anti-Patterns to Avoid
1. **Redundant conditions** like `(Amount > 0) or (Amount < 0)` instead of `Amount <> 0`
2. **Comparing booleans to true/false** like `if (IsValid = true)` instead of `if IsValid`
3. **Complex negative conditions** that can be simplified with De Morgan's laws
4. **Mixing different validation concerns** in single boolean expressions

## Related Concepts
- **Guard Clauses**: Using early returns to simplify validation logic
- **Specification Pattern**: Encapsulating business rules in reusable boolean expressions
- **Chain of Responsibility**: Breaking complex validation into sequential checks
- **Short-Circuit Evaluation**: Leveraging AL's boolean evaluation order for performance