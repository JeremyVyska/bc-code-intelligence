# No. Series Module BC24+ Implementation Examples

## Basic Number Generation Patterns

### Document Number Generation (Sales Order)
```al
// BC24+ approach using new No. Series module
trigger OnInsert()
var
    NoSeries: Codeunit "No. Series";
begin
    if "No." = '' then
        "No." := NoSeries.GetNextNo("No. Series", WorkDate());
end;
```

### Manual Number Validation
```al
// BC24+ manual number handling
procedure ValidateManualNumber()
var
    NoSeries: Codeunit "No. Series";
begin
    NoSeries.TestManual("No. Series");
    // Built-in error handling provides detailed messages
end;
```

### Batch Number Allocation
```al
// BC24+ batch operations using NoSeriesBatch
procedure AllocateDocumentNumbers(DocumentCount: Integer): List of [Code[20]]
var
    NoSeriesBatch: Codeunit "No. Series - Batch";
    NumberList: List of [Code[20]];
begin
    NumberList := NoSeriesBatch.GetNextNos("No. Series", WorkDate(), DocumentCount);
    exit(NumberList);
end;
```

## Advanced Integration Patterns

### Series Relationship Management
```al
// BC24+ relationship validation
procedure ValidateSeriesRelationship(NewSeries: Code[20])
var
    NoSeries: Codeunit "No. Series";
begin
    if not NoSeries.AreRelated("No. Series", NewSeries) then
        Error('Series %1 and %2 are not related', "No. Series", NewSeries);
end;
```

### Number Lookup with User Selection
```al
// BC24+ series lookup functionality
procedure SelectRelatedSeries()
var
    NoSeries: Codeunit "No. Series";
    SelectedSeries: Code[20];
begin
    SelectedSeries := "No. Series";
    if NoSeries.LookupRelatedNoSeries(SelectedSeries, "No. Series") then
        Validate("No. Series", SelectedSeries);
end;
```

### Error Handling with Try Methods
```al
// BC24+ error handling approach
procedure SafeNumberGeneration(): Boolean
var
    NoSeries: Codeunit "No. Series";
    GeneratedNo: Code[20];
begin
    if NoSeries.TryGetNextNo(GeneratedNo, "No. Series", WorkDate()) then begin
        "No." := GeneratedNo;
        exit(true);
    end else begin
        HandleNumberGenerationError();
        exit(false);
    end;
end;
```