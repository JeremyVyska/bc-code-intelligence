# Table Event Batch Operation Impact - AL Code Samples

## Batch Processing with Event Management

```al
codeunit 50300 "Batch Processing Manager"
{
    var
        BatchMode: Boolean;
        BatchSize: Integer;
        ProcessedCount: Integer;

    procedure ProcessItemsBatch(var Item: Record Item)
    begin
        BatchMode := true;
        BatchSize := 0;
        ProcessedCount := 0;
        
        if Item.FindSet() then
            repeat
                ProcessedCount += 1;
                UpdateItemData(Item);
                
                if ProcessedCount mod 100 = 0 then
                    Commit();
                    
            until Item.Next() = 0;
        
        BatchMode := false;
    end;
    
    [EventSubscriber(ObjectType::Table, Database::Item, 'OnAfterModifyEvent', '', false, false)]
    local procedure OnAfterItemModify(var Rec: Record Item; var xRec: Record Item; RunTrigger: Boolean)
    begin
        if BatchMode then
            exit; // Skip processing during batch operations
            
        UpdateItemStatistics(Rec);
    end;
}
```

## Optimized Batch Operations

```al
procedure ProcessSalesLinesBatch(var SalesHeader: Record "Sales Header")
var
    SalesLine: Record "Sales Line";
    TempSalesLine: Record "Sales Line" temporary;
begin
    SalesLine.SetRange("Document Type", SalesHeader."Document Type");
    SalesLine.SetRange("Document No.", SalesHeader."No.");
    
    // Load into temporary table to minimize event firing
    if SalesLine.FindSet() then
        repeat
            TempSalesLine := SalesLine;
            TempSalesLine.Insert();
        until SalesLine.Next() = 0;
    
    // Process without triggering events
    if TempSalesLine.FindSet() then
        repeat
            ProcessSalesLineData(TempSalesLine);
        until TempSalesLine.Next() = 0;
end;
```

## Event Subscriber Performance Optimization

```al
codeunit 50301 "Performance Optimized Subscriber"
{
    var
        ProcessingEnabled: Boolean;
        
    [EventSubscriber(ObjectType::Table, Database::"Sales Line", 'OnAfterInsertEvent', '', false, false)]
    local procedure OnAfterSalesLineInsert(var Rec: Record "Sales Line"; RunTrigger: Boolean)
    begin
        if not RunTrigger then exit;
        if not ProcessingEnabled then exit;
        
        // Minimal processing for performance
        UpdateSalesStatistics(Rec);
    end;
    
    procedure EnableProcessing()
    begin
        ProcessingEnabled := true;
    end;
    
    procedure DisableProcessing()
    begin
        ProcessingEnabled := false;
    end;
}
```