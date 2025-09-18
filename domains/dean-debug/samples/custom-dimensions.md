# Custom Dimension Implementation Examples - AL Code Samples

## Standard Custom Dimensions Pattern
```al
codeunit 50110 "Custom Dimensions Examples"
{
    // Example 1: Building comprehensive custom dimensions
    procedure BuildStandardDimensions(): Dictionary of [Text, Text]
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        // Core identification dimensions
        CustomDimensions.Add('CompanyName', CompanyName());
        CustomDimensions.Add('UserID', UserId());
        CustomDimensions.Add('SessionID', Format(SessionId()));
        
        // Environment context
        CustomDimensions.Add('Environment', GetEnvironmentType());
        CustomDimensions.Add('AppVersion', GetAppVersion());
        CustomDimensions.Add('BCVersion', GetBCVersion());
        
        // Temporal dimensions
        CustomDimensions.Add('Timestamp', Format(CurrentDateTime, 0, 9));
        CustomDimensions.Add('Date', Format(Today(), 0, '<Year4>-<Month,2>-<Day,2>'));
        CustomDimensions.Add('Hour', Format(Time(), 0, '<Hours24>'));
        
        exit(CustomDimensions);
    end;

    // Example 2: Business context dimensions
    procedure BuildBusinessContextDimensions(CustomerNo: Code[20]; ItemNo: Code[20]): Dictionary of [Text, Text]
    var
        CustomDimensions: Dictionary of [Text, Text];
        Customer: Record Customer;
        Item: Record Item;
    begin
        CustomDimensions := BuildStandardDimensions();
        
        // Customer context
        if Customer.Get(CustomerNo) then begin
            CustomDimensions.Add('CustomerNo', CustomerNo);
            CustomDimensions.Add('CustomerGroup', Customer."Customer Posting Group");
            CustomDimensions.Add('SalespersonCode', Customer."Salesperson Code");
            CustomDimensions.Add('CountryRegion', Customer."Country/Region Code");
        end;
        
        // Item context
        if Item.Get(ItemNo) then begin
            CustomDimensions.Add('ItemNo', ItemNo);
            CustomDimensions.Add('ItemCategory', Item."Item Category Code");
            CustomDimensions.Add('ItemType', Format(Item.Type));
            CustomDimensions.Add('UnitOfMeasure', Item."Base Unit of Measure");
        end;
        
        exit(CustomDimensions);
    end;
    
    // Helper methods
    local procedure GetEnvironmentType(): Text
    begin
        exit('Production'); // Implement actual environment detection
    end;
    
    local procedure GetAppVersion(): Text
    begin
        exit('1.0.0.0'); // Implement actual version detection
    end;
    
    local procedure GetBCVersion(): Text
    begin
        exit('24.0'); // Implement actual BC version detection
    end;
}
```

## Performance-Optimized Dimension Patterns
```al
codeunit 50111 "Optimized Dimension Builder"
{
    var
        CachedDimensions: Dictionary of [Text, Text];
        LastCacheTime: DateTime;
        CacheValidityMinutes: Integer;

    // Example 3: Cached dimension building for performance
    procedure GetCachedStandardDimensions(): Dictionary of [Text, Text]
    begin
        if ShouldRefreshCache() then
            RefreshCachedDimensions();
            
        exit(CachedDimensions);
    end;

    local procedure ShouldRefreshCache(): Boolean
    begin
        if LastCacheTime = 0DT then
            exit(true);
            
        exit(CurrentDateTime - LastCacheTime > CacheValidityMinutes * 60000);
    end;

    local procedure RefreshCachedDimensions()
    begin
        CachedDimensions.Clear();
        
        // Cache static dimensions that don't change frequently
        CachedDimensions.Add('CompanyName', CompanyName());
        CachedDimensions.Add('Environment', 'Production');
        CachedDimensions.Add('AppVersion', '1.0.0.0');
        CachedDimensions.Add('BCVersion', '24.0');
        
        LastCacheTime := CurrentDateTime;
    end;

    // Example 4: Conditional dimension building
    procedure BuildConditionalDimensions(IncludeDetailed: Boolean; IncludePerformance: Boolean): Dictionary of [Text, Text]
    var
        CustomDimensions: Dictionary of [Text, Text];
    begin
        CustomDimensions := GetCachedStandardDimensions();
        
        // Always include dynamic dimensions
        CustomDimensions.Add('UserID', UserId());
        CustomDimensions.Add('Timestamp', Format(CurrentDateTime, 0, 9));
        
        if IncludeDetailed then begin
            CustomDimensions.Add('SessionID', Format(SessionId()));
            CustomDimensions.Add('ClientType', Format(ClientType()));
            CustomDimensions.Add('LanguageID', Format(GlobalLanguage()));
        end;
        
        if IncludePerformance then begin
            CustomDimensions.Add('MemoryUsage', 'Unknown');
            CustomDimensions.Add('CPUTime', 'Unknown');
        end;
        
        exit(CustomDimensions);
    end;
}
```

## Domain-Specific Dimension Patterns
```al
codeunit 50112 "Domain Telemetry Dimensions"
{
    // Example 5: Sales process dimensions
    procedure BuildSalesDimensions(SalesHeader: Record "Sales Header"): Dictionary of [Text, Text]
    var
        CustomDimensions: Dictionary of [Text, Text];
        Customer: Record Customer;
        Salesperson: Record "Salesperson/Purchaser";
    begin
        CustomDimensions.Add('Domain', 'Sales');
        CustomDimensions.Add('DocumentType', Format(SalesHeader."Document Type"));
        CustomDimensions.Add('DocumentNo', SalesHeader."No.");
        CustomDimensions.Add('CustomerNo', SalesHeader."Sell-to Customer No.");
        
        if Customer.Get(SalesHeader."Sell-to Customer No.") then begin
            CustomDimensions.Add('CustomerGroup', Customer."Customer Posting Group");
            CustomDimensions.Add('PaymentTerms', Customer."Payment Terms Code");
            CustomDimensions.Add('CustomerSize', ClassifyCustomerSize(Customer."Credit Limit (LCY)"));
        end;
        
        if Salesperson.Get(SalesHeader."Salesperson Code") then
            CustomDimensions.Add('SalespersonName', Salesperson.Name);
            
        // Financial dimensions
        CustomDimensions.Add('CurrencyCode', SalesHeader."Currency Code");
        CustomDimensions.Add('AmountLCY', Format(SalesHeader."Amount Including VAT"));
        CustomDimensions.Add('PriceGroup', SalesHeader."Customer Price Group");
        
        exit(CustomDimensions);
    end;

    // Helper function for customer classification
    local procedure ClassifyCustomerSize(CreditLimit: Decimal): Text
    begin
        case true of
            CreditLimit >= 1000000:
                exit('Enterprise');
            CreditLimit >= 100000:
                exit('Large');
            CreditLimit >= 10000:
                exit('Medium');
            else
                exit('Small');
        end;
    end;
}
```