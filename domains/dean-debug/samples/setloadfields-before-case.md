# SetLoadFields Placement Before Case Statements - AL Code Samples

## Basic Case Statement Placement
```al
codeunit 50400 "Case Statement Placement"
{
    procedure CorrectCaseStatementPattern()
    var
        SalesHeader: Record "Sales Header";
    begin
        // CORRECT: SetLoadFields before case statement evaluation
        SalesHeader.SetLoadFields("Document Type", "No.", "Sell-to Customer Name", "Amount Including VAT");
        SalesHeader.SetRange(Status, SalesHeader.Status::Open);
        
        if SalesHeader.FindSet() then
            repeat
                // Case statement after SetLoadFields - all needed fields already loaded
                case SalesHeader."Document Type" of
                    SalesHeader."Document Type"::Quote:
                        ProcessQuote(SalesHeader."No.", SalesHeader."Sell-to Customer Name");
                    SalesHeader."Document Type"::Order:
                        ProcessOrder(SalesHeader."No.", SalesHeader."Sell-to Customer Name", SalesHeader."Amount Including VAT");
                    SalesHeader."Document Type"::Invoice:
                        ProcessInvoice(SalesHeader."No.", SalesHeader."Amount Including VAT");
                    SalesHeader."Document Type"::"Credit Memo":
                        ProcessCreditMemo(SalesHeader."No.", SalesHeader."Amount Including VAT");
                end;
            until SalesHeader.Next() = 0;
    end;
    
    procedure IncorrectCaseStatementPattern()
    var
        SalesHeader: Record "Sales Header";
    begin
        // INCORRECT: SetLoadFields inside case statement branches
        SalesHeader.SetRange(Status, SalesHeader.Status::Open);
        
        if SalesHeader.FindSet() then
            repeat
                case SalesHeader."Document Type" of
                    SalesHeader."Document Type"::Quote:
                        begin
                            // SetLoadFields called too late - Document Type field already accessed for case evaluation
                            SalesHeader.SetLoadFields("No.", "Sell-to Customer Name");
                            ProcessQuote(SalesHeader."No.", SalesHeader."Sell-to Customer Name");
                        end;
                    SalesHeader."Document Type"::Order:
                        begin
                            SalesHeader.SetLoadFields("No.", "Sell-to Customer Name", "Amount Including VAT");
                            ProcessOrder(SalesHeader."No.", SalesHeader."Sell-to Customer Name", SalesHeader."Amount Including VAT");
                        end;
                end;
            until SalesHeader.Next() = 0;
    end;

    local procedure ProcessQuote(DocumentNo: Code[20]; CustomerName: Text[100])
    begin
        // Process quote
    end;
    
    local procedure ProcessOrder(DocumentNo: Code[20]; CustomerName: Text[100]; Amount: Decimal)
    begin
        // Process order
    end;
    
    local procedure ProcessInvoice(DocumentNo: Code[20]; Amount: Decimal)
    begin
        // Process invoice
    end;
    
    local procedure ProcessCreditMemo(DocumentNo: Code[20]; Amount: Decimal)
    begin
        // Process credit memo
    end;
}
```

## Advanced Case Statement Patterns
```al
codeunit 50401 "Advanced Case Placement"
{
    procedure ComplexCaseStatementOptimization()
    var
        SalesLine: Record "Sales Line";
    begin
        // Load all fields that might be needed by any case branch
        SalesLine.SetLoadFields("Document Type", "Document No.", "Line No.", "Type", "No.", 
                               "Quantity", "Unit Price", "Line Amount");
        SalesLine.SetRange("Document Type", SalesLine."Document Type"::Order);
        
        if SalesLine.FindSet() then
            repeat
                // Primary case statement for line type
                case SalesLine.Type of
                    SalesLine.Type::Item:
                        ProcessItemLine(SalesLine);
                    SalesLine.Type::Resource:
                        ProcessResourceLine(SalesLine);
                    SalesLine.Type::"G/L Account":
                        ProcessGLAccountLine(SalesLine);
                    SalesLine.Type::"Charge (Item)":
                        ProcessItemChargeLine(SalesLine);
                    else
                        HandleUnknownLineType(SalesLine);
                end;
            until SalesLine.Next() = 0;
    end;
    
    procedure NestedCaseStatementExample()
    var
        Item: Record Item;
    begin
        // SetLoadFields covers all fields needed by nested case statements
        Item.SetLoadFields("No.", "Type", "Item Category Code", "Unit Price", "Inventory", "Blocked");
        Item.SetRange("No.", '1000', '9999');
        
        if Item.FindSet() then
            repeat
                // Outer case statement
                case Item.Type of
                    Item.Type::Inventory,
                    Item.Type::"Non-Inventory":
                        begin
                            // Inner case statement - fields already loaded
                            case Item."Item Category Code" of
                                'CHAIR':
                                    ProcessChairItem(Item."No.", Item."Unit Price");
                                'TABLE':
                                    ProcessTableItem(Item."No.", Item.Inventory);
                                'DESK':
                                    ProcessDeskItem(Item."No.", Item."Unit Price", Item.Inventory);
                                else
                                    ProcessGeneralItem(Item."No.");
                            end;
                        end;
                    Item.Type::Service:
                        ProcessServiceItem(Item."No.", Item."Unit Price");
                end;
            until Item.Next() = 0;
    end;

    procedure ConditionalCaseStatements()
    var
        Customer: Record Customer;
        ProcessHighPriority: Boolean;
    begin
        // SetLoadFields accounts for all possible case statement branches
        Customer.SetLoadFields("No.", "Name", "Customer Posting Group", "Credit Limit (LCY)", 
                               "Balance (LCY)", "Phone No.");
        
        ProcessHighPriority := Time > 120000T; // After noon, process high priority
        
        if Customer.FindSet() then
            repeat
                if ProcessHighPriority then begin
                    // High priority case statement
                    case Customer."Customer Posting Group" of
                        'VIP':
                            if Customer."Credit Limit (LCY)" > 100000 then
                                ProcessVIPCustomer(Customer."No.", Customer.Name, Customer."Credit Limit (LCY)");
                        'CORPORATE':
                            ProcessCorporateCustomer(Customer."No.", Customer."Balance (LCY)");
                    end;
                end else begin
                    // Standard priority case statement
                    case Customer."Customer Posting Group" of
                        'DOMESTIC':
                            ProcessDomesticCustomer(Customer."No.", Customer.Name);
                        'FOREIGN':
                            ProcessForeignCustomer(Customer."No.", Customer."Phone No.");
                    end;
                end;
            until Customer.Next() = 0;
    end;

    // Helper procedures for line processing
    local procedure ProcessItemLine(var SalesLine: Record "Sales Line")
    begin
        // Item line processing using pre-loaded fields
        Message('Processing Item Line: %1, Qty: %2', SalesLine."No.", SalesLine.Quantity);
    end;
    
    local procedure ProcessResourceLine(var SalesLine: Record "Sales Line")
    begin
        // Resource line processing
        Message('Processing Resource Line: %1, Amount: %2', SalesLine."No.", SalesLine."Line Amount");
    end;
    
    local procedure ProcessGLAccountLine(var SalesLine: Record "Sales Line")
    begin
        // G/L Account line processing
        Message('Processing G/L Account Line: %1', SalesLine."No.");
    end;
    
    local procedure ProcessItemChargeLine(var SalesLine: Record "Sales Line")
    begin
        // Item charge processing
        Message('Processing Item Charge Line: %1', SalesLine."No.");
    end;
    
    local procedure HandleUnknownLineType(var SalesLine: Record "Sales Line")
    begin
        Error('Unknown line type for document %1, line %2', SalesLine."Document No.", SalesLine."Line No.");
    end;

    local procedure ProcessChairItem(ItemNo: Code[20]; UnitPrice: Decimal)
    begin
        Message('Chair Item: %1, Price: %2', ItemNo, UnitPrice);
    end;

    local procedure ProcessTableItem(ItemNo: Code[20]; Inventory: Decimal)
    begin
        Message('Table Item: %1, Inventory: %2', ItemNo, Inventory);
    end;

    local procedure ProcessDeskItem(ItemNo: Code[20]; UnitPrice: Decimal; Inventory: Decimal)
    begin
        Message('Desk Item: %1, Price: %2, Inventory: %3', ItemNo, UnitPrice, Inventory);
    end;

    local procedure ProcessGeneralItem(ItemNo: Code[20])
    begin
        Message('General Item: %1', ItemNo);
    end;

    local procedure ProcessServiceItem(ItemNo: Code[20]; UnitPrice: Decimal)
    begin
        Message('Service Item: %1, Price: %2', ItemNo, UnitPrice);
    end;

    local procedure ProcessVIPCustomer(CustomerNo: Code[20]; Name: Text[100]; CreditLimit: Decimal)
    begin
        Message('VIP Customer: %1 - %2, Credit Limit: %3', CustomerNo, Name, CreditLimit);
    end;

    local procedure ProcessCorporateCustomer(CustomerNo: Code[20]; Balance: Decimal)
    begin
        Message('Corporate Customer: %1, Balance: %2', CustomerNo, Balance);
    end;

    local procedure ProcessDomesticCustomer(CustomerNo: Code[20]; Name: Text[100])
    begin
        Message('Domestic Customer: %1 - %2', CustomerNo, Name);
    end;

    local procedure ProcessForeignCustomer(CustomerNo: Code[20]; PhoneNo: Text[30])
    begin
        Message('Foreign Customer: %1, Phone: %2', CustomerNo, PhoneNo);
    end;
}
```

## Performance Impact Analysis
```al
codeunit 50402 "Case Statement Performance"
{
    procedure DemonstratePerformanceImpact()
    var
        Item: Record Item;
        StartTime: DateTime;
        EndTime: DateTime;
    begin
        // Scenario 1: SetLoadFields after case statement evaluation (inefficient)
        StartTime := CurrentDateTime();
        Item.SetRange("Item Category Code", 'CHAIR');
        
        if Item.FindSet() then
            repeat
                // Item Category Code accessed before SetLoadFields - loads all fields
                case Item."Item Category Code" of
                    'CHAIR':
                        begin
                            Item.SetLoadFields("No.", "Unit Price"); // Too late
                            ProcessChairItem(Item."No.", Item."Unit Price");
                        end;
                    'TABLE':
                        begin
                            Item.SetLoadFields("No.", "Inventory"); // Too late
                            ProcessTableItem(Item."No.", Item.Inventory);
                        end;
                end;
            until Item.Next() = 0;
        EndTime := CurrentDateTime();
        Message('Inefficient approach: %1 ms', EndTime - StartTime);
        
        // Scenario 2: SetLoadFields before case statement evaluation (efficient)
        StartTime := CurrentDateTime();
        Item.Reset();
        Item.SetLoadFields("Item Category Code", "No.", "Unit Price", "Inventory"); // Before case evaluation
        Item.SetRange("Item Category Code", 'CHAIR');
        
        if Item.FindSet() then
            repeat
                // Item Category Code already loaded efficiently
                case Item."Item Category Code" of
                    'CHAIR':
                        ProcessChairItem(Item."No.", Item."Unit Price");
                    'TABLE':
                        ProcessTableItem(Item."No.", Item.Inventory);
                end;
            until Item.Next() = 0;
        EndTime := CurrentDateTime();
        Message('Efficient approach: %1 ms', EndTime - StartTime);
    end;
    
    local procedure ProcessChairItem(ItemNo: Code[20]; UnitPrice: Decimal)
    begin
        // Process chair item
    end;
    
    local procedure ProcessTableItem(ItemNo: Code[20]; Inventory: Decimal)
    begin
        // Process table item
    end;
}
```