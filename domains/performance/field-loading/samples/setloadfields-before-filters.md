# SetLoadFields Placement Before Filters - AL Code Samples

## Basic Before Filter Placement
```al
codeunit 50300 "Before Filter Placement"
{
    procedure CorrectPlacementExample()
    var
        SalesHeader: Record "Sales Header";
    begin
        // CORRECT: SetLoadFields called before any filters
        SalesHeader.SetLoadFields("No.", "Sell-to Customer Name", "Amount Including VAT");
        
        // Filters applied after SetLoadFields
        SalesHeader.SetRange("Sell-to Customer No.", '10000', '50000');
        SalesHeader.SetRange("Document Date", WorkDate() - 30, WorkDate());
        SalesHeader.SetRange(Status, SalesHeader.Status::Open);
        
        if SalesHeader.FindSet() then
            repeat
                ProcessOrder(SalesHeader."No.", SalesHeader."Sell-to Customer Name", SalesHeader."Amount Including VAT");
            until SalesHeader.Next() = 0;
    end;
    
    procedure IncorrectPlacementExample()
    var
        SalesHeader: Record "Sales Header";
    begin
        // INCORRECT: Filters applied before SetLoadFields
        SalesHeader.SetRange("Sell-to Customer No.", '10000', '50000');
        SalesHeader.SetRange("Document Date", WorkDate() - 30, WorkDate());
        
        // SetLoadFields called after filters - may not be fully effective
        SalesHeader.SetLoadFields("No.", "Sell-to Customer Name", "Amount Including VAT");
        SalesHeader.SetRange(Status, SalesHeader.Status::Open);
        
        if SalesHeader.FindSet() then
            repeat
                ProcessOrder(SalesHeader."No.", SalesHeader."Sell-to Customer Name", SalesHeader."Amount Including VAT");
            until SalesHeader.Next() = 0;
    end;

    procedure MultipleFilterSetExample()
    var
        Item: Record Item;
    begin
        // SetLoadFields must be first, before any filtering operations
        Item.SetLoadFields("No.", "Description", "Unit Price", "Inventory");
        
        // Multiple filter operations - all after SetLoadFields
        Item.SetRange("No.", '1000', '9999');
        Item.SetFilter("Last Date Modified", '>=%1', CalcDate('-6M', WorkDate()));
        Item.SetFilter("Item Category Code", 'CHAIR|TABLE|DESK');
        Item.SetFilter("Unit Price", '>=%1', 50);
        
        if Item.FindSet() then
            repeat
                ProcessInventoryItem(Item."No.", Item.Description, Item."Unit Price", Item.Inventory);
            until Item.Next() = 0;
    end;

    local procedure ProcessOrder(OrderNo: Code[20]; CustomerName: Text[100]; Amount: Decimal)
    begin
        // Process order data
    end;

    local procedure ProcessInventoryItem(ItemNo: Code[20]; Description: Text[100]; UnitPrice: Decimal; Inventory: Decimal)
    begin
        // Process inventory item
    end;
}
```

## Advanced Placement Patterns
```al
codeunit 50301 "Advanced Filter Placement"
{
    procedure ConditionalFilteringExample(CustomerNo: Code[20]; IncludeShipped: Boolean)
    var
        SalesHeader: Record "Sales Header";
    begin
        // SetLoadFields always first, regardless of conditional logic
        SalesHeader.SetLoadFields("No.", "Sell-to Customer Name", "Amount Including VAT", "Shipment Date");
        
        // Base filters
        SalesHeader.SetRange("Sell-to Customer No.", CustomerNo);
        SalesHeader.SetRange("Document Date", WorkDate() - 90, WorkDate());
        
        // Conditional filters - still after SetLoadFields
        if not IncludeShipped then
            SalesHeader.SetRange("Completely Shipped", false);
            
        if SalesHeader.FindSet() then
            repeat
                ProcessCustomerOrder(SalesHeader."No.", SalesHeader."Sell-to Customer Name", 
                                   SalesHeader."Amount Including VAT", SalesHeader."Shipment Date");
            until SalesHeader.Next() = 0;
    end;
    
    procedure ResetAndRefilterExample()
    var
        Customer: Record Customer;
    begin
        // Initial setup with SetLoadFields first
        Customer.SetLoadFields("No.", "Name", "Credit Limit (LCY)");
        Customer.SetRange("No.", '10000', '30000');
        
        ProcessCustomerSet(Customer, 'First batch');
        
        // When resetting filters, SetLoadFields should be called again if changing field selection
        Customer.Reset();
        Customer.SetLoadFields("No.", "Name", "Phone No."); // Different fields
        Customer.SetRange("No.", '40000', '60000');
        
        ProcessCustomerSet(Customer, 'Second batch');
    end;

    procedure DynamicFilteringExample()
    var
        Item: Record Item;
        FilterCategoryCode: Code[20];
        FilterBlocked: Boolean;
        i: Integer;
    begin
        // SetLoadFields established before any dynamic filtering
        Item.SetLoadFields("No.", "Description", "Item Category Code", "Unit Price");
        
        // Dynamic filtering loop - filters change but SetLoadFields remains effective
        for i := 1 to 3 do begin
            case i of
                1: FilterCategoryCode := 'CHAIR';
                2: FilterCategoryCode := 'TABLE';
                3: FilterCategoryCode := 'DESK';
            end;
            FilterBlocked := i mod 2 = 0; // Alternate blocked status
            
            Item.Reset();
            Item.SetRange("Item Category Code", FilterCategoryCode);
            Item.SetRange(Blocked, FilterBlocked);
            
            ProcessItemSet(Item, StrSubstNo('Category %1', FilterCategoryCode));
        end;
    end;

    local procedure ProcessCustomerOrder(OrderNo: Code[20]; CustomerName: Text[100]; Amount: Decimal; ShipmentDate: Date)
    begin
        // Process customer order
    end;
    
    local procedure ProcessCustomerSet(var Customer: Record Customer; BatchName: Text)
    begin
        if Customer.FindSet() then
            repeat
                // Process customer based on loaded fields
                Message('Processing %1: Customer %2', BatchName, Customer."No.");
            until Customer.Next() = 0;
    end;

    local procedure ProcessItemSet(var Item: Record Item; BatchName: Text)
    begin
        if Item.FindSet() then
            repeat
                // Process item based on loaded fields
                Message('Processing %1: Item %2 - %3', BatchName, Item."No.", Item.Description);
            until Item.Next() = 0;
    end;
}
```

## Common Placement Mistakes
```al
codeunit 50302 "Filter Placement Mistakes"
{
    procedure MistakeExample_FiltersFirst()
    var
        Customer: Record Customer;
    begin
        // MISTAKE: Applying filters before SetLoadFields
        Customer.SetRange("No.", '10000');
        Customer.SetRange("Customer Posting Group", 'DOMESTIC');
        
        // SetLoadFields less effective when called after filters
        Customer.SetLoadFields("No.", "Name", "Credit Limit (LCY)");
        
        // This works but is suboptimal
        if Customer.FindSet() then
            repeat
                ProcessCustomer(Customer."No.", Customer.Name, Customer."Credit Limit (LCY)");
            until Customer.Next() = 0;
    end;
    
    procedure BestPracticePattern()
    var
        Customer: Record Customer;
    begin
        // BEST PRACTICE: Clear separation of concerns
        // 1. Field loading configuration
        Customer.SetLoadFields("No.", "Name", "Credit Limit (LCY)", "Phone No.");
        
        // 2. Data filtering
        Customer.SetRange("No.", '10000', '50000');
        Customer.SetFilter("Last Date Modified", '>=%1', CalcDate('-3M', WorkDate()));
        Customer.SetRange("Customer Posting Group", 'DOMESTIC');
        
        // 3. Data processing
        if Customer.FindSet() then
            repeat
                ProcessCustomerWithPhone(Customer."No.", Customer.Name, 
                                        Customer."Credit Limit (LCY)", Customer."Phone No.");
            until Customer.Next() = 0;
    end;

    local procedure ProcessCustomer(CustomerNo: Code[20]; Name: Text[100]; CreditLimit: Decimal)
    begin
        // Process basic customer data
    end;

    local procedure ProcessCustomerWithPhone(CustomerNo: Code[20]; Name: Text[100]; CreditLimit: Decimal; PhoneNo: Text[30])
    begin
        // Process customer with phone information
    end;
}
```