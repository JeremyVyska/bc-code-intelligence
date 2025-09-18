# BC24 No. Series Conversion Examples

## Complete Conversion Examples

### Sales Document Conversion
```al
// BEFORE (BC14-23): Legacy NoSeriesManagement approach
table 50100 "Sales Document"
{
    trigger OnInsert()
    var
        NoSeriesMgt: Codeunit NoSeriesManagement;
    begin
        if "No." = '' then
            NoSeriesMgt.InitSeries(GetNoSeriesCode(), xRec."No. Series", 0D, "No.", "No. Series");
    end;

    procedure GetNoSeriesCode(): Code[20]
    begin
        exit('SALES-DOC');
    end;
}

// AFTER (BC24+): New No. Series module approach
table 50100 "Sales Document"
{
    trigger OnInsert()
    var
        NoSeries: Codeunit "No. Series";
    begin
        if "No." = '' then
            "No." := NoSeries.GetNextNo("No. Series", WorkDate());
    end;

    procedure GetNoSeriesCode(): Code[20]
    begin
        exit('SALES-DOC');
    end;
}
```

### Manual Number Testing Conversion
```al
// BEFORE (BC14-23): Multiple method calls for manual testing
procedure ValidateManualEntry()
var
    NoSeriesMgt: Codeunit NoSeriesManagement;
begin
    NoSeriesMgt.TestManual("No. Series");
    if not NoSeriesMgt.ManualNoAllowed("No. Series") then
        Error('Manual numbers not allowed for series %1', "No. Series");
end;

// AFTER (BC24+): Single method with built-in logic
procedure ValidateManualEntry()
var
    NoSeries: Codeunit "No. Series";
begin
    NoSeries.TestManual("No. Series");
    // Built-in error handling provides comprehensive validation
end;
```

### Number Peek vs Allocation Conversion
```al
// BEFORE (BC14-23): Boolean parameter controls behavior
procedure GetNumberPreview(): Code[20]
var
    NoSeriesMgt: Codeunit NoSeriesManagement;
begin
    exit(NoSeriesMgt.GetNextNo("No. Series", WorkDate(), false)); // Peek only
end;

procedure AllocateNumber(): Code[20]
var
    NoSeriesMgt: Codeunit NoSeriesManagement;
begin
    exit(NoSeriesMgt.GetNextNo("No. Series", WorkDate(), true)); // Allocate
end;

// AFTER (BC24+): Explicit method names for clear intent
procedure GetNumberPreview(): Code[20]
var
    NoSeries: Codeunit "No. Series";
begin
    exit(NoSeries.PeekNextNo("No. Series", WorkDate())); // Explicit peek
end;

procedure AllocateNumber(): Code[20]
var
    NoSeries: Codeunit "No. Series";
begin
    exit(NoSeries.GetNextNo("No. Series", WorkDate())); // Explicit allocation
end;
```

## Event Migration Examples

### Replacing Obsolete Events
```al
// BEFORE (BC14-23): Event subscriber approach
[EventSubscriber(ObjectType::Codeunit, Codeunit::NoSeriesManagement, 'OnBeforeGetNextNo', '', true, true)]
local procedure OnBeforeGetNextNo(NoSeriesCode: Code[20]; var NoSeriesLine: Record "No. Series Line"; var IsHandled: Boolean)
begin
    // Custom logic before number generation
    if NoSeriesCode = 'SPECIAL-SERIES' then begin
        PerformCustomValidation(NoSeriesCode);
        IsHandled := true;
    end;
end;

// AFTER (BC24+): Direct method call with wrapper
procedure GetNextNoWithCustomLogic(SeriesCode: Code[20]): Code[20]
var
    NoSeries: Codeunit "No. Series";
begin
    // Pre-processing logic (replaces OnBeforeGetNextNo)
    if SeriesCode = 'SPECIAL-SERIES' then
        PerformCustomValidation(SeriesCode);

    // Get number using new module
    exit(NoSeries.GetNextNo(SeriesCode, WorkDate()));
end;
```

### Complex Event Replacement
```al
// BEFORE (BC14-23): Multiple event subscriptions
[EventSubscriber(ObjectType::Codeunit, Codeunit::NoSeriesManagement, 'OnAfterGetNextNo', '', true, true)]
local procedure OnAfterGetNextNo(NoSeriesCode: Code[20]; NextNo: Code[20])
begin
    LogNumberAllocation(NoSeriesCode, NextNo);
    UpdateRelatedSeries(NoSeriesCode, NextNo);
end;

// AFTER (BC24+): Integrated procedure with post-processing
procedure AllocateNumberWithLogging(SeriesCode: Code[20]): Code[20]
var
    NoSeries: Codeunit "No. Series";
    AllocatedNumber: Code[20];
begin
    // Allocate number
    AllocatedNumber := NoSeries.GetNextNo(SeriesCode, WorkDate());

    // Post-processing logic (replaces OnAfterGetNextNo)
    LogNumberAllocation(SeriesCode, AllocatedNumber);
    UpdateRelatedSeries(SeriesCode, AllocatedNumber);

    exit(AllocatedNumber);
end;
```

## Complete Migration Checklist

### Step 1: Variable Declaration Updates
```al
// Remove these legacy declarations
var
    NoSeriesMgt: Codeunit NoSeriesManagement;

// Replace with new module references
var
    NoSeries: Codeunit "No. Series";
    NoSeriesBatch: Codeunit "No. Series - Batch";
```

### Step 2: Method Call Replacements
```al
// Replace all instances of these patterns:
NoSeriesMgt.InitSeries(...) → NoSeries.GetNextNo(...)
NoSeriesMgt.TestManual(...) → NoSeries.TestManual(...)
NoSeriesMgt.SelectSeries(...) → NoSeries.LookupRelatedNoSeries(...)
NoSeriesMgt.GetNextNo(..., false) → NoSeries.PeekNextNo(...)
NoSeriesMgt.GetNextNo(..., true) → NoSeries.GetNextNo(...)
```

### Step 3: Error Handling Migration
```al
// BEFORE: Custom error handling required
if not NoSeriesMgt.TryFunction(...) then
    Error('Custom error message');

// AFTER: Built-in error handling with try methods
if not NoSeries.TryGetNextNo(ResultNo, SeriesCode, WorkDate()) then
    HandleSpecificFailureScenario();
```