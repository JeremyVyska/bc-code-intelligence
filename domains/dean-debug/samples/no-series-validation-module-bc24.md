# No. Series Validation Module BC24+ Examples

## Core Validation Patterns

### Number Format Validation
```al
// BC24+ format validation using built-in methods
procedure ValidateNumberFormat(NumberToValidate: Code[20]; SeriesCode: Code[20]): Boolean
var
    NoSeries: Codeunit "No. Series";
begin
    exit(NoSeries.IsValidNo(NumberToValidate, SeriesCode));
end;
```

### Series Configuration Validation
```al
// BC24+ series existence and configuration validation
procedure ValidateSeriesConfiguration(SeriesCode: Code[20])
var
    NoSeries: Codeunit "No. Series";
begin
    NoSeries.VerifySeriesExists(SeriesCode);
    // Automatic detailed error if series doesn't exist or is misconfigured
end;
```

### Date-Based Validation
```al
// BC24+ date validation for number series
procedure ValidateSeriesForDate(SeriesCode: Code[20]; ValidateDate: Date): Boolean
var
    NoSeries: Codeunit "No. Series";
begin
    exit(NoSeries.IsValidForDate(SeriesCode, ValidateDate));
end;
```

## Advanced Validation Scenarios

### Batch Allocation Validation
```al
// BC24+ validate batch allocation feasibility
procedure ValidateBatchAllocation(SeriesCode: Code[20]; RequiredCount: Integer): Boolean
var
    NoSeriesBatch: Codeunit "No. Series - Batch";
begin
    exit(NoSeriesBatch.CanAllocateNumbers(SeriesCode, RequiredCount));
end;
```

### Cross-Series Validation
```al
// BC24+ validate consistency across related series
procedure ValidateRelatedSeriesConsistency(PrimarySeries: Code[20]; SecondarySeries: Code[20])
var
    NoSeries: Codeunit "No. Series";
begin
    if not NoSeries.AreRelated(PrimarySeries, SecondarySeries) then
        Error('Series %1 and %2 are not properly related', PrimarySeries, SecondarySeries);

    // Additional validation for related series
    NoSeries.ValidateRelatedSeries(PrimarySeries, SecondarySeries);
end;
```

### Transaction-Safe Validation
```al
// BC24+ validation within transaction context
procedure ValidateWithinTransaction(SeriesCode: Code[20]; TransactionDate: Date)
var
    NoSeries: Codeunit "No. Series";
begin
    NoSeries.ValidateForTransaction(SeriesCode, TransactionDate);
    // Built-in validation considers transaction boundaries
end;
```

## Error Handling and Recovery

### Graceful Validation Failure Handling
```al
// BC24+ handle validation failures gracefully
procedure HandleValidationFailure(SeriesCode: Code[20]): Boolean
var
    NoSeries: Codeunit "No. Series";
    ValidationResult: Record "Validation Result";
begin
    ValidationResult := NoSeries.ValidateSeriesConfiguration(SeriesCode);
    if not ValidationResult.Success then begin
        ProcessValidationErrors(ValidationResult);
        exit(false);
    end;
    exit(true);
end;
```

### Automatic Recovery Patterns
```al
// BC24+ automatic recovery from validation issues
procedure RecoverFromValidationFailure(SeriesCode: Code[20])
var
    NoSeries: Codeunit "No. Series";
begin
    if not NoSeries.TryGetNextNo("No.", SeriesCode, WorkDate()) then begin
        // Attempt alternative series or recovery procedure
        TryAlternativeSeriesAllocation(SeriesCode);
    end;
end;
```