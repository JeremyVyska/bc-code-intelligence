# DeleteAll Alternative Implementation Patterns

## Conditional DeleteAll vs Direct SQL
```al
procedure CleanupTempData(TableID: Integer; BusinessLogicRequired: Boolean)
var
    TempDataBuffer: Record "Temp Data Buffer";
begin
    TempDataBuffer.SetRange("Source Table ID", TableID);
    
    if BusinessLogicRequired then begin
        // Use DeleteAll when OnDelete triggers must execute
        TempDataBuffer.DeleteAll(true);
    end else begin
        // Use direct deletion for simple cleanup
        if TempDataBuffer.FindSet() then
            TempDataBuffer.DeleteAll(false);
    end;
end;
```

## Batch Processing with Size Limits
```al
procedure ProcessLargeDataDeletion(var RecordToDelete: Record "Large Data Table")
var
    BatchSize: Integer;
    ProcessedCount: Integer;
begin
    BatchSize := 1000;
    ProcessedCount := 0;
    
    RecordToDelete.SetCurrentKey("Created Date Time");
    if RecordToDelete.FindSet() then
        repeat
            RecordToDelete.Delete(true); // Individual delete with trigger execution
            ProcessedCount += 1;
            
            // Commit periodically to avoid large transactions
            if ProcessedCount mod BatchSize = 0 then begin
                Commit();
                Message('Processed %1 records...', ProcessedCount);
            end;
        until RecordToDelete.Next() = 0;
end;
```

## Hybrid Approach with Validation
```al
procedure SmartDeleteWithValidation(var OrderLine: Record "Sales Line")
var
    SalesHeader: Record "Sales Header";
    HasComplexRelationships: Boolean;
begin
    // Analyze data complexity before choosing deletion method
    HasComplexRelationships := CheckForReservations(OrderLine) or 
                              CheckForItemTracking(OrderLine) or
                              CheckForJobRelations(OrderLine);
    
    if HasComplexRelationships then begin
        // Use DeleteAll with trigger execution for complex scenarios
        OrderLine.DeleteAll(true);
    end else begin
        // Use faster approach for simple scenarios
        if OrderLine.FindSet() then begin
            SalesHeader.Get(OrderLine."Document Type", OrderLine."Document No.");
            OrderLine.DeleteAll(false);
            // Manual header update since OnDelete didn't run
            SalesHeader.Modify(true);
        end;
    end;
end;
```

## Performance-Optimized Cleanup
```al
procedure OptimizedCleanupRoutine()
var
    LogEntry: Record "Activity Log";
    CutoffDate: Date;
    RecordCount: Integer;
begin
    CutoffDate := CalcDate('-1Y', WorkDate());
    
    // First pass: Count records for performance estimation
    LogEntry.SetFilter("Activity Date", '<%1', CutoffDate);
    RecordCount := LogEntry.Count();
    
    case RecordCount of
        0..999:
            // Small dataset: Use DeleteAll with business logic
            LogEntry.DeleteAll(true);
        1000..9999:
            // Medium dataset: Use DeleteAll without triggers
            LogEntry.DeleteAll(false);
        else begin
            // Large dataset: Process in batches
            ProcessLargeDeletionBatch(LogEntry);
        end;
    end;
end;

local procedure ProcessLargeDeletionBatch(var LogEntry: Record "Activity Log")
var
    BatchCounter: Integer;
begin
    if LogEntry.FindSet() then
        repeat
            LogEntry.Delete(false);
            BatchCounter += 1;
            
            if BatchCounter mod 5000 = 0 then begin
                Commit();
                Sleep(100); // Brief pause to reduce system load
            end;
        until LogEntry.Next() = 0;
end;
```