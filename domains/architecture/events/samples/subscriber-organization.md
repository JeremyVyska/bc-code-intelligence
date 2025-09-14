# Subscriber Codeunit Size Optimization - AL Code Samples

## Anti-Pattern: Monolithic Subscriber

```al
codeunit 50500 "Monolithic Event Subscriber"
{
    // ANTI-PATTERN: All subscribers in one large codeunit
    
    [EventSubscriber(ObjectType::Table, Database::Customer, 'OnBeforeInsertEvent', '', false, false)]
    local procedure OnBeforeCustomerInsert(var Rec: Record Customer; RunTrigger: Boolean)
    begin
        ValidateCustomerData(Rec);
        CheckCustomerCreditLimit(Rec);
        UpdateCustomerStatistics(Rec);
        LogCustomerCreation(Rec);
        // Many more operations creating a massive procedure
    end;

    [EventSubscriber(ObjectType::Table, Database::Item, 'OnBeforeInsertEvent', '', false, false)]
    local procedure OnBeforeItemInsert(var Rec: Record Item; RunTrigger: Boolean)
    begin
        ValidateItemData(Rec);
        SetupItemDefaults(Rec);
        CheckItemInventory(Rec);
        // Many more operations
    end;
}
```

## Optimized Pattern: Focused Subscribers

```al
codeunit 50501 "Customer Event Subscriber"
{
    // Single responsibility: Handle only customer-related events
    
    [EventSubscriber(ObjectType::Table, Database::Customer, 'OnBeforeInsertEvent', '', false, false)]
    local procedure OnBeforeCustomerInsert(var Rec: Record Customer; RunTrigger: Boolean)
    begin
        if not RunTrigger then exit;
        
        CustomerValidationHandler.ValidateNewCustomer(Rec);
        CustomerSetupHandler.ApplyDefaultSettings(Rec);
        CustomerNotificationHandler.NotifyNewCustomer(Rec);
    end;
    
    var
        CustomerValidationHandler: Codeunit "Customer Validation Handler";
        CustomerSetupHandler: Codeunit "Customer Setup Handler";
        CustomerNotificationHandler: Codeunit "Customer Notification Handler";
}

codeunit 50502 "Item Event Subscriber"
{
    // Single responsibility: Handle only item-related events
    
    [EventSubscriber(ObjectType::Table, Database::Item, 'OnBeforeInsertEvent', '', false, false)]
    local procedure OnBeforeItemInsert(var Rec: Record Item; RunTrigger: Boolean)
    begin
        if not RunTrigger then exit;
        
        ItemValidationHandler.ValidateNewItem(Rec);
        ItemSetupHandler.ApplyDefaultSettings(Rec);
    end;
    
    var
        ItemValidationHandler: Codeunit "Item Validation Handler";
        ItemSetupHandler: Codeunit "Item Setup Handler";
}
```

## Specialized Handler Pattern

```al
codeunit 50510 "Customer Validation Handler"
{
    // Focused on customer validation logic only
    
    procedure ValidateNewCustomer(var Customer: Record Customer)
    begin
        ValidateRequiredFields(Customer);
        ValidateBusinessRules(Customer);
        ValidateDataIntegrity(Customer);
    end;
    
    local procedure ValidateRequiredFields(var Customer: Record Customer)
    begin
        if Customer.Name = '' then
            Error('Customer name is required');
            
        if Customer."Country/Region Code" = '' then
            Error('Country/Region Code is required');
    end;
    
    local procedure ValidateBusinessRules(var Customer: Record Customer)
    begin
        if (Customer."Credit Limit (LCY)" > 100000) and (Customer."Payment Terms Code" = '') then
            Error('Payment terms required for customers with credit limit over 100,000');
    end;
}
```