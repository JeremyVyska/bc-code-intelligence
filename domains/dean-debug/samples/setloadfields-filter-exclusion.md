# SetLoadFields Filter Field Exclusion - AL Code Samples

## Basic Filter Field Exclusion
```al
codeunit 50200 "Filter Field Exclusion Examples"
{
    procedure ExcludeFilterFieldsBasic()
    var
        Customer: Record Customer;
    begin
        // Exclude filter fields from loading - they're used for filtering only
        Customer.SetLoadFields("Name", "Credit Limit (LCY)", "Phone No.");
        
        // Filter fields (No., Customer Posting Group) not loaded - filtering uses cached values
        Customer.SetRange("No.", '10000', '20000');
        Customer.SetRange("Customer Posting Group", 'DOMESTIC');
        Customer.SetRange("Blocked", Customer.Blocked::" ");
        
        if Customer.FindSet() then
            repeat
                // Only non-filter fields loaded, improving performance
                ProcessCustomerData(Customer.Name, Customer."Credit Limit (LCY)", Customer."Phone No.");
            until Customer.Next() = 0;
    end;

    procedure ComplexFilterExclusion()
    var
        Item: Record Item;
    begin
        // Complex filtering scenario - exclude all filter fields from loading
        Item.SetLoadFields("Description", "Unit Price", "Inventory");
        
        // Multiple filter fields - all excluded from SetLoadFields
        Item.SetRange("No.", '1000', '9999');
        Item.SetFilter("Item Category Code", 'CHAIR|TABLE|DESK');
        Item.SetRange("Blocked", false);
        Item.SetFilter("Unit Price", '>%1', 100);
        Item.SetRange("Type", Item.Type::Inventory);
        
        if Item.FindSet() then
            repeat
                // Filter fields available for validation but not loaded from database
                if ShouldProcessItem(Item."No.", Item."Item Category Code") then
                    ProcessItemData(Item.Description, Item."Unit Price", Item.Inventory);
            until Item.Next() = 0;
    end;

    procedure ConditionalFilterExclusion(IncludeFilterFields: Boolean)
    var
        SalesLine: Record "Sales Line";
    begin
        if IncludeFilterFields then
            // When filter fields needed for processing, include them
            SalesLine.SetLoadFields("Document No.", "Line No.", "Type", "No.", "Quantity", "Unit Price")
        else
            // When filter fields not needed for processing, exclude them
            SalesLine.SetLoadFields("Quantity", "Unit Price");
            
        SalesLine.SetRange("Document Type", SalesLine."Document Type"::Order);
        SalesLine.SetRange("Document No.", 'SO001');
        SalesLine.SetRange("Type", SalesLine.Type::Item);
        
        if SalesLine.FindSet() then
            repeat
                if IncludeFilterFields then
                    ProcessWithFilterData(SalesLine."Document No.", SalesLine."No.", SalesLine.Quantity)
                else
                    ProcessWithoutFilterData(SalesLine.Quantity, SalesLine."Unit Price");
            until SalesLine.Next() = 0;
    end;

    local procedure ProcessCustomerData(Name: Text[100]; CreditLimit: Decimal; PhoneNo: Text[30])
    begin
        // Process loaded customer data
    end;

    local procedure ShouldProcessItem(ItemNo: Code[20]; CategoryCode: Code[20]): Boolean
    begin
        exit((ItemNo <> '') and (CategoryCode <> ''));
    end;

    local procedure ProcessItemData(Description: Text[100]; UnitPrice: Decimal; Inventory: Decimal)
    begin
        // Process non-filter item data
    end;

    local procedure ProcessWithFilterData(DocumentNo: Code[20]; ItemNo: Code[20]; Quantity: Decimal)
    begin
        // Process including filter field data
    end;

    local procedure ProcessWithoutFilterData(Quantity: Decimal; UnitPrice: Decimal)
    begin
        // Process excluding filter field data
    end;
}
```

## Performance Impact Examples
```al
codeunit 50201 "Filter Exclusion Performance"
{
    procedure DemonstratePerformanceImpact()
    var
        Customer: Record Customer;
        StartTime: DateTime;
        EndTime: DateTime;
    begin
        // Scenario 1: Including filter fields unnecessarily
        StartTime := CurrentDateTime();
        Customer.SetLoadFields(); // Loads all fields including filters
        Customer.SetRange("No.", '10000', '50000');
        Customer.SetRange("Customer Posting Group", 'DOMESTIC');
        
        if Customer.FindSet() then
            repeat
                // Only using non-filter fields but all were loaded
                ProcessCustomerSales(Customer.Name, Customer."Credit Limit (LCY)");
            until Customer.Next() = 0;
        EndTime := CurrentDateTime();
        LogPerformance('With filter fields loaded', EndTime - StartTime);
        
        // Scenario 2: Excluding filter fields appropriately
        StartTime := CurrentDateTime();
        Customer.Reset();
        Customer.SetLoadFields("Name", "Credit Limit (LCY)"); // Exclude filter fields
        Customer.SetRange("No.", '10000', '50000');
        Customer.SetRange("Customer Posting Group", 'DOMESTIC');
        
        if Customer.FindSet() then
            repeat
                // Only non-filter fields loaded, better performance
                ProcessCustomerSales(Customer.Name, Customer."Credit Limit (LCY)");
            until Customer.Next() = 0;
        EndTime := CurrentDateTime();
        LogPerformance('With filter fields excluded', EndTime - StartTime);
    end;

    local procedure ProcessCustomerSales(Name: Text[100]; CreditLimit: Decimal)
    begin
        // Process customer sales data
    end;

    local procedure LogPerformance(Scenario: Text; Duration: Duration)
    begin
        Message('%1: %2 ms', Scenario, Duration);
    end;
}
```