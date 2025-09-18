# SetLoadFields Primary Key Optimization - AL Code Samples

## Basic Primary Key Optimization
```al
codeunit 50100 "SetLoadFields PK Examples"
{
    procedure OptimizedPrimaryKeyLookup(DocumentType: Enum "Sales Document Type"; DocumentNo: Code[20])
    var
        SalesHeader: Record "Sales Header";
    begin
        // Load only primary key fields for existence checks
        SalesHeader.SetLoadFields("Document Type", "No.");
        SalesHeader.SetRange("Document Type", DocumentType);
        SalesHeader.SetRange("No.", DocumentNo);
        
        if SalesHeader.FindSet() then
            repeat
                // Primary key fields already loaded, minimal memory usage
                ProcessDocumentHeader(SalesHeader."Document Type", SalesHeader."No.");
            until SalesHeader.Next() = 0;
    end;

    procedure ConditionalPrimaryKeyLoading(CustomerNo: Code[20]; LoadFullData: Boolean)
    var
        Customer: Record Customer;
    begin
        if LoadFullData then
            // Load all fields when full processing needed
            Customer.SetLoadFields()
        else
            // Minimal primary key loading for counting/existence
            Customer.SetLoadFields("No.");
            
        Customer.SetRange("No.", CustomerNo);
        
        if Customer.FindFirst() then
            if LoadFullData then
                ProcessFullCustomer(Customer)
            else
                LogCustomerExists(Customer."No.");
    end;

    procedure BulkPrimaryKeyProcessing()
    var
        Item: Record Item;
        ItemNumbers: List of [Code[20]];
        ItemNo: Code[20];
    begin
        // First pass: collect primary keys only
        Item.SetLoadFields("No.");
        if Item.FindSet() then
            repeat
                ItemNumbers.Add(Item."No.");
            until Item.Next() = 0;
            
        // Second pass: process specific records with full data
        foreach ItemNo in ItemNumbers do begin
            Item.SetLoadFields(); // Load all fields for processing
            if Item.Get(ItemNo) then
                ProcessFullItem(Item);
        end;
    end;

    local procedure ProcessDocumentHeader(DocumentType: Enum "Sales Document Type"; DocumentNo: Code[20])
    begin
        // Processing logic using only primary key data
    end;

    local procedure ProcessFullCustomer(var Customer: Record Customer)
    begin
        // Full customer processing
    end;

    local procedure LogCustomerExists(CustomerNo: Code[20])
    begin
        // Minimal logging using primary key data
    end;

    local procedure ProcessFullItem(var Item: Record Item)
    begin
        // Full item processing
    end;
}
```

## Performance Comparison Examples
```al
codeunit 50101 "PK Performance Comparison"
{
    procedure UnoptimizedPrimaryKeyAccess()
    var
        Customer: Record Customer;
    begin
        // BAD: Loading all fields when only checking existence
        Customer.SetRange("Customer Posting Group", 'DOMESTIC');
        
        if Customer.FindSet() then
            repeat
                // Only using primary key fields but all fields were loaded
                if IsValidCustomer(Customer."No.") then
                    ProcessValidCustomer(Customer."No.");
            until Customer.Next() = 0;
    end;
    
    procedure OptimizedPrimaryKeyAccess()
    var
        Customer: Record Customer;
    begin
        // GOOD: Loading only required primary key fields
        Customer.SetLoadFields("No.");
        Customer.SetRange("Customer Posting Group", 'DOMESTIC');
        
        if Customer.FindSet() then
            repeat
                // Minimal memory usage, faster iteration
                if IsValidCustomer(Customer."No.") then
                    ProcessValidCustomer(Customer."No.");
            until Customer.Next() = 0;
    end;

    procedure SmartPrimaryKeyWithLazyLoading()
    var
        Customer: Record Customer;
        FullDataCustomer: Record Customer;
    begin
        // BEST: Minimal loading with selective full data loading
        Customer.SetLoadFields("No.");
        Customer.SetRange("Customer Posting Group", 'DOMESTIC');
        
        if Customer.FindSet() then
            repeat
                if IsValidCustomer(Customer."No.") then begin
                    // Load full data only when needed
                    if FullDataCustomer.Get(Customer."No.") then
                        ProcessFullCustomer(FullDataCustomer);
                end;
            until Customer.Next() = 0;
    end;

    local procedure IsValidCustomer(CustomerNo: Code[20]): Boolean
    begin
        exit(CustomerNo <> '');
    end;

    local procedure ProcessValidCustomer(CustomerNo: Code[20])
    begin
        // Processing using only primary key data
    end;

    local procedure ProcessFullCustomer(var Customer: Record Customer)
    begin
        // Full customer processing
    end;
}
```