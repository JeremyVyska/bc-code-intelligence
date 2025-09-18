# No Series Implementation Patterns - AL Code Examples

This sample demonstrates implementing robust number generation and sequence management in Business Central.

## Basic No Series Implementation

```al
// Custom number generation codeunit
codeunit 50500 "Custom No. Series Management"
{
    procedure GetNextNo(NoSeriesCode: Code[20]; SeriesDate: Date): Code[20]
    var
        NoSeries: Record "No. Series";
        NoSeriesLine: Record "No. Series Line";
        NextNo: Code[20];
    begin
        if not NoSeries.Get(NoSeriesCode) then
            Error('No. Series %1 does not exist', NoSeriesCode);

        NoSeriesLine := FindApplicableNoSeriesLine(NoSeriesCode, SeriesDate);
        NextNo := GenerateNextNumber(NoSeriesLine);

        UpdateNoSeriesLine(NoSeriesLine, NextNo);
        LogNumberGeneration(NoSeriesCode, NextNo);

        exit(NextNo);
    end;

    procedure GetNextNoWithGapManagement(NoSeriesCode: Code[20]; SeriesDate: Date; FillGaps: Boolean): Code[20]
    var
        NextNo: Code[20];
        GapNo: Code[20];
    begin
        if FillGaps then begin
            GapNo := FindFirstGap(NoSeriesCode);
            if GapNo <> '' then begin
                MarkGapAsFilled(NoSeriesCode, GapNo);
                exit(GapNo);
            end;
        end;

        exit(GetNextNo(NoSeriesCode, SeriesDate));
    end;

    procedure CreateCustomNoSeries(NoSeriesCode: Code[20]; Description: Text[100]; StartingNo: Code[20]; EndingNo: Code[20]): Boolean
    var
        NoSeries: Record "No. Series";
        NoSeriesLine: Record "No. Series Line";
    begin
        // Create No. Series header
        if NoSeries.Get(NoSeriesCode) then
            exit(false); // Already exists

        NoSeries.Init();
        NoSeries.Code := NoSeriesCode;
        NoSeries.Description := Description;
        NoSeries."Default Nos." := true;
        NoSeries."Manual Nos." := false;
        NoSeries.Insert(true);

        // Create No. Series line
        NoSeriesLine.Init();
        NoSeriesLine."Series Code" := NoSeriesCode;
        NoSeriesLine."Line No." := 10000;
        NoSeriesLine."Starting Date" := Today;
        NoSeriesLine."Starting No." := StartingNo;
        NoSeriesLine."Ending No." := EndingNo;
        NoSeriesLine."Last No. Used" := '';
        NoSeriesLine."Increment-by No." := 1;
        NoSeriesLine.Insert(true);

        exit(true);
    end;

    local procedure FindApplicableNoSeriesLine(NoSeriesCode: Code[20]; SeriesDate: Date): Record "No. Series Line"
    var
        NoSeriesLine: Record "No. Series Line";
    begin
        NoSeriesLine.SetRange("Series Code", NoSeriesCode);
        NoSeriesLine.SetFilter("Starting Date", '<=%1', SeriesDate);
        NoSeriesLine.SetFilter("Ending Date", '%1|>=%2', 0D, SeriesDate);

        if not NoSeriesLine.FindLast() then
            Error('No applicable No. Series Line found for %1 on %2', NoSeriesCode, SeriesDate);

        exit(NoSeriesLine);
    end;

    local procedure GenerateNextNumber(var NoSeriesLine: Record "No. Series Line"): Code[20]
    var
        NextNo: Code[20];
        NumberPart: BigInteger;
        PrefixPart: Text;
        SuffixPart: Text;
        NumberLength: Integer;
    begin
        if NoSeriesLine."Last No. Used" = '' then
            NextNo := NoSeriesLine."Starting No."
        else begin
            // Parse current number
            ParseNumberComponents(NoSeriesLine."Last No. Used", PrefixPart, NumberPart, SuffixPart, NumberLength);

            // Increment number
            NumberPart += NoSeriesLine."Increment-by No.";

            // Reconstruct number
            NextNo := ReconstructNumber(PrefixPart, NumberPart, SuffixPart, NumberLength);
        end;

        // Validate against ending number
        if (NoSeriesLine."Ending No." <> '') and (NextNo > NoSeriesLine."Ending No.") then
            Error('No. Series %1 has reached its ending number %2', NoSeriesLine."Series Code", NoSeriesLine."Ending No.");

        exit(NextNo);
    end;

    local procedure UpdateNoSeriesLine(var NoSeriesLine: Record "No. Series Line"; NextNo: Code[20])
    begin
        NoSeriesLine."Last No. Used" := NextNo;
        NoSeriesLine."Last Date Used" := Today;
        NoSeriesLine.Modify(true);
    end;

    local procedure ParseNumberComponents(NumberStr: Code[20]; var Prefix: Text; var Number: BigInteger; var Suffix: Text; var NumberLength: Integer)
    var
        i: Integer;
        CurrentChar: Char;
        NumberStartPos: Integer;
        NumberEndPos: Integer;
        NumberPart: Text;
    begin
        Clear(Prefix);
        Clear(Suffix);
        Number := 0;
        NumberLength := 0;
        NumberStartPos := 0;
        NumberEndPos := 0;

        // Find the start of the numeric part
        for i := 1 to StrLen(NumberStr) do begin
            CurrentChar := NumberStr[i];
            if CurrentChar in ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'] then begin
                if NumberStartPos = 0 then
                    NumberStartPos := i;
                NumberEndPos := i;
            end else begin
                if NumberStartPos > 0 then
                    break;
            end;
        end;

        if NumberStartPos > 0 then begin
            // Extract components
            if NumberStartPos > 1 then
                Prefix := CopyStr(NumberStr, 1, NumberStartPos - 1);

            NumberPart := CopyStr(NumberStr, NumberStartPos, NumberEndPos - NumberStartPos + 1);
            NumberLength := StrLen(NumberPart);
            Evaluate(Number, NumberPart);

            if NumberEndPos < StrLen(NumberStr) then
                Suffix := CopyStr(NumberStr, NumberEndPos + 1);
        end;
    end;

    local procedure ReconstructNumber(Prefix: Text; Number: BigInteger; Suffix: Text; NumberLength: Integer): Code[20]
    var
        NumberPart: Text;
        Result: Text;
    begin
        NumberPart := Format(Number);

        // Pad with leading zeros if necessary
        while StrLen(NumberPart) < NumberLength do
            NumberPart := '0' + NumberPart;

        Result := Prefix + NumberPart + Suffix;
        exit(CopyStr(Result, 1, 20));
    end;

    local procedure FindFirstGap(NoSeriesCode: Code[20]): Code[20]
    var
        NoSeriesGap: Record "No. Series Gap";
    begin
        NoSeriesGap.SetRange("Series Code", NoSeriesCode);
        NoSeriesGap.SetRange(Filled, false);
        if NoSeriesGap.FindFirst() then
            exit(NoSeriesGap."Gap No.");

        exit('');
    end;

    local procedure MarkGapAsFilled(NoSeriesCode: Code[20]; GapNo: Code[20])
    var
        NoSeriesGap: Record "No. Series Gap";
    begin
        if NoSeriesGap.Get(NoSeriesCode, GapNo) then begin
            NoSeriesGap.Filled := true;
            NoSeriesGap."Fill Date" := Today;
            NoSeriesGap."Fill Time" := Time;
            NoSeriesGap.Modify(true);
        end;
    end;

    local procedure LogNumberGeneration(NoSeriesCode: Code[20]; GeneratedNo: Code[20])
    var
        NoSeriesLog: Record "No. Series Log";
    begin
        NoSeriesLog.Init();
        NoSeriesLog."Entry No." := GetNextLogEntryNo();
        NoSeriesLog."Series Code" := NoSeriesCode;
        NoSeriesLog."Generated No." := GeneratedNo;
        NoSeriesLog."Generation Date" := Today;
        NoSeriesLog."Generation Time" := Time;
        NoSeriesLog."User ID" := UserId;
        NoSeriesLog.Insert(true);
    end;

    local procedure GetNextLogEntryNo(): Integer
    var
        NoSeriesLog: Record "No. Series Log";
    begin
        if NoSeriesLog.FindLast() then
            exit(NoSeriesLog."Entry No." + 1)
        else
            exit(1);
    end;
}
```

## Concurrency-Safe Number Generation

```al
// High-performance number generation with concurrency control
codeunit 50501 "Concurrent No. Series Management"
{
    procedure GetNextNoSafe(NoSeriesCode: Code[20]; SeriesDate: Date): Code[20]
    var
        NoSeriesLine: Record "No. Series Line";
        NextNo: Code[20];
        RetryCount: Integer;
        MaxRetries: Integer;
    begin
        MaxRetries := 10;
        RetryCount := 0;

        repeat
            RetryCount += 1;

            if TryGetNextNoWithLock(NoSeriesCode, SeriesDate, NextNo) then
                exit(NextNo);

            if RetryCount >= MaxRetries then
                Error('Unable to generate next number after %1 attempts. Series may be in use by another user.', MaxRetries);

            // Wait before retry (exponential backoff)
            Sleep(RetryCount * 100);

        until false;
    end;

    [TryFunction]
    local procedure TryGetNextNoWithLock(NoSeriesCode: Code[20]; SeriesDate: Date; var NextNo: Code[20])
    var
        NoSeriesLine: Record "No. Series Line";
        NoSeriesLock: Record "No. Series Lock";
        LockAcquired: Boolean;
    begin
        // Attempt to acquire lock
        LockAcquired := TryAcquireSeriesLock(NoSeriesCode);
        if not LockAcquired then
            Error('Lock not acquired');

        try
            // Find applicable line
            NoSeriesLine := FindApplicableNoSeriesLine(NoSeriesCode, SeriesDate);

            // Generate next number
            NextNo := GenerateNextNumberSafe(NoSeriesLine);

            // Update line
            UpdateNoSeriesLineSafe(NoSeriesLine, NextNo);

            // Commit changes
            Commit();

        finally
            // Always release lock
            ReleaseSeriesLock(NoSeriesCode);
        end;
    end;

    [TryFunction]
    local procedure TryAcquireSeriesLock(NoSeriesCode: Code[20]): Boolean
    var
        NoSeriesLock: Record "No. Series Lock";
        LockTimeout: DateTime;
    begin
        LockTimeout := CurrentDateTime + 5000; // 5 second timeout

        NoSeriesLock.SetRange("Series Code", NoSeriesCode);

        repeat
            if not NoSeriesLock.FindFirst() then begin
                // Create lock record
                NoSeriesLock.Init();
                NoSeriesLock."Series Code" := NoSeriesCode;
                NoSeriesLock."Locked By" := UserId;
                NoSeriesLock."Lock Time" := CurrentDateTime;
                NoSeriesLock.Insert(true);
                exit(true);
            end;

            // Check if lock is stale (older than 1 minute)
            if NoSeriesLock."Lock Time" < (CurrentDateTime - 60000) then begin
                NoSeriesLock.Delete(true);
                continue;
            end;

            Sleep(50); // Wait 50ms before retry

        until CurrentDateTime > LockTimeout;

        exit(false);
    end;

    local procedure ReleaseSeriesLock(NoSeriesCode: Code[20])
    var
        NoSeriesLock: Record "No. Series Lock";
    begin
        NoSeriesLock.SetRange("Series Code", NoSeriesCode);
        NoSeriesLock.SetRange("Locked By", UserId);
        if NoSeriesLock.FindFirst() then
            NoSeriesLock.Delete(true);
    end;

    local procedure FindApplicableNoSeriesLine(NoSeriesCode: Code[20]; SeriesDate: Date): Record "No. Series Line"
    var
        NoSeriesLine: Record "No. Series Line";
    begin
        NoSeriesLine.SetRange("Series Code", NoSeriesCode);
        NoSeriesLine.SetFilter("Starting Date", '<=%1', SeriesDate);
        NoSeriesLine.SetFilter("Ending Date", '%1|>=%2', 0D, SeriesDate);

        if not NoSeriesLine.FindLast() then
            Error('No applicable No. Series Line found for %1 on %2', NoSeriesCode, SeriesDate);

        exit(NoSeriesLine);
    end;

    local procedure GenerateNextNumberSafe(var NoSeriesLine: Record "No. Series Line"): Code[20]
    var
        NextNo: Code[20];
        LastUsedNo: Code[20];
    begin
        // Re-read to get latest values
        NoSeriesLine.Find();

        if NoSeriesLine."Last No. Used" = '' then
            NextNo := NoSeriesLine."Starting No."
        else begin
            NextNo := IncrementNumber(NoSeriesLine."Last No. Used", NoSeriesLine."Increment-by No.");
        end;

        // Validate against ending number
        if (NoSeriesLine."Ending No." <> '') and (NextNo > NoSeriesLine."Ending No.") then
            Error('No. Series %1 has reached its ending number %2', NoSeriesLine."Series Code", NoSeriesLine."Ending No.");

        exit(NextNo);
    end;

    local procedure UpdateNoSeriesLineSafe(var NoSeriesLine: Record "No. Series Line"; NextNo: Code[20])
    begin
        NoSeriesLine."Last No. Used" := NextNo;
        NoSeriesLine."Last Date Used" := Today;
        NoSeriesLine.Modify(true);
    end;

    local procedure IncrementNumber(CurrentNo: Code[20]; IncrementBy: Integer): Code[20]
    var
        NumberPart: BigInteger;
        PrefixPart: Text;
        SuffixPart: Text;
        NumberLength: Integer;
    begin
        ParseNumberComponents(CurrentNo, PrefixPart, NumberPart, SuffixPart, NumberLength);
        NumberPart += IncrementBy;
        exit(ReconstructNumber(PrefixPart, NumberPart, SuffixPart, NumberLength));
    end;

    local procedure ParseNumberComponents(NumberStr: Code[20]; var Prefix: Text; var Number: BigInteger; var Suffix: Text; var NumberLength: Integer)
    begin
        // Implementation from previous example
    end;

    local procedure ReconstructNumber(Prefix: Text; Number: BigInteger; Suffix: Text; NumberLength: Integer): Code[20]
    begin
        // Implementation from previous example
    end;
}
```

## Batch Number Allocation

```al
// Optimized batch allocation for high-volume scenarios
codeunit 50502 "Batch No. Series Allocation"
{
    var
        AllocatedNumbers: Dictionary of [Code[20], List of [Code[20]]];

    procedure AllocateNumberBatch(NoSeriesCode: Code[20]; BatchSize: Integer): List of [Code[20]]
    var
        NoSeriesLine: Record "No. Series Line";
        AllocatedBatch: List of [Code[20]];
        NextNo: Code[20];
        i: Integer;
    begin
        if BatchSize <= 0 then
            Error('Batch size must be greater than zero');

        if BatchSize > 1000 then
            Error('Batch size cannot exceed 1000');

        // Lock the series for batch allocation
        LockNoSeriesForBatch(NoSeriesCode);

        try
            NoSeriesLine := FindApplicableNoSeriesLine(NoSeriesCode, Today);

            // Allocate batch
            for i := 1 to BatchSize do begin
                NextNo := GenerateNextNumberInBatch(NoSeriesLine, i);
                AllocatedBatch.Add(NextNo);
            end;

            // Update the line with the last allocated number
            UpdateNoSeriesLineForBatch(NoSeriesLine, AllocatedBatch.Get(BatchSize));

            // Cache allocated numbers
            AllocatedNumbers.Set(NoSeriesCode, AllocatedBatch);

            Commit();

        finally
            UnlockNoSeriesForBatch(NoSeriesCode);
        end;

        exit(AllocatedBatch);
    end;

    procedure GetNextFromBatch(NoSeriesCode: Code[20]): Code[20]
    var
        BatchList: List of [Code[20]];
        NextNo: Code[20];
    begin
        if not AllocatedNumbers.Get(NoSeriesCode, BatchList) then
            Error('No allocated batch found for series %1', NoSeriesCode);

        if BatchList.Count = 0 then begin
            // Batch exhausted, allocate new batch
            BatchList := AllocateNumberBatch(NoSeriesCode, 100); // Default batch size
        end;

        NextNo := BatchList.Get(1);
        BatchList.RemoveAt(1);
        AllocatedNumbers.Set(NoSeriesCode, BatchList);

        LogBatchUsage(NoSeriesCode, NextNo);

        exit(NextNo);
    end;

    procedure GetBatchStatus(NoSeriesCode: Code[20]): Dictionary of [Text, Variant]
    var
        Status: Dictionary of [Text, Variant];
        BatchList: List of [Code[20]];
    begin
        Status.Set('SeriesCode', NoSeriesCode);

        if AllocatedNumbers.Get(NoSeriesCode, BatchList) then begin
            Status.Set('HasBatch', true);
            Status.Set('RemainingNumbers', BatchList.Count);
            if BatchList.Count > 0 then begin
                Status.Set('NextAvailable', BatchList.Get(1));
                Status.Set('LastInBatch', BatchList.Get(BatchList.Count));
            end;
        end else begin
            Status.Set('HasBatch', false);
            Status.Set('RemainingNumbers', 0);
        end;

        exit(Status);
    end;

    local procedure FindApplicableNoSeriesLine(NoSeriesCode: Code[20]; SeriesDate: Date): Record "No. Series Line"
    var
        NoSeriesLine: Record "No. Series Line";
    begin
        NoSeriesLine.SetRange("Series Code", NoSeriesCode);
        NoSeriesLine.SetFilter("Starting Date", '<=%1', SeriesDate);
        NoSeriesLine.SetFilter("Ending Date", '%1|>=%2', 0D, SeriesDate);

        if not NoSeriesLine.FindLast() then
            Error('No applicable No. Series Line found for %1 on %2', NoSeriesCode, SeriesDate);

        exit(NoSeriesLine);
    end;

    local procedure GenerateNextNumberInBatch(var NoSeriesLine: Record "No. Series Line"; BatchIndex: Integer): Code[20]
    var
        BaseNumber: Code[20];
        NumberPart: BigInteger;
        PrefixPart: Text;
        SuffixPart: Text;
        NumberLength: Integer;
    begin
        if BatchIndex = 1 then begin
            if NoSeriesLine."Last No. Used" = '' then
                BaseNumber := NoSeriesLine."Starting No."
            else
                BaseNumber := IncrementNumber(NoSeriesLine."Last No. Used", NoSeriesLine."Increment-by No.");
        end else begin
            BaseNumber := IncrementNumber(BaseNumber, NoSeriesLine."Increment-by No.");
        end;

        // Validate against ending number
        if (NoSeriesLine."Ending No." <> '') and (BaseNumber > NoSeriesLine."Ending No.") then
            Error('No. Series %1 would exceed its ending number %2 in batch allocation', NoSeriesLine."Series Code", NoSeriesLine."Ending No.");

        exit(BaseNumber);
    end;

    local procedure UpdateNoSeriesLineForBatch(var NoSeriesLine: Record "No. Series Line"; LastAllocatedNo: Code[20])
    begin
        NoSeriesLine."Last No. Used" := LastAllocatedNo;
        NoSeriesLine."Last Date Used" := Today;
        NoSeriesLine.Modify(true);
    end;

    local procedure LockNoSeriesForBatch(NoSeriesCode: Code[20])
    var
        NoSeriesBatchLock: Record "No. Series Batch Lock";
    begin
        NoSeriesBatchLock.Init();
        NoSeriesBatchLock."Series Code" := NoSeriesCode;
        NoSeriesBatchLock."Locked By" := UserId;
        NoSeriesBatchLock."Lock Time" := CurrentDateTime;
        NoSeriesBatchLock.Insert(true);
    end;

    local procedure UnlockNoSeriesForBatch(NoSeriesCode: Code[20])
    var
        NoSeriesBatchLock: Record "No. Series Batch Lock";
    begin
        NoSeriesBatchLock.SetRange("Series Code", NoSeriesCode);
        NoSeriesBatchLock.SetRange("Locked By", UserId);
        if NoSeriesBatchLock.FindFirst() then
            NoSeriesBatchLock.Delete(true);
    end;

    local procedure IncrementNumber(CurrentNo: Code[20]; IncrementBy: Integer): Code[20]
    begin
        // Implementation from previous examples
    end;

    local procedure LogBatchUsage(NoSeriesCode: Code[20]; UsedNo: Code[20])
    var
        NoSeriesBatchLog: Record "No. Series Batch Log";
    begin
        NoSeriesBatchLog.Init();
        NoSeriesBatchLog."Entry No." := GetNextBatchLogEntryNo();
        NoSeriesBatchLog."Series Code" := NoSeriesCode;
        NoSeriesBatchLog."Used No." := UsedNo;
        NoSeriesBatchLog."Usage Date" := Today;
        NoSeriesBatchLog."Usage Time" := Time;
        NoSeriesBatchLog."User ID" := UserId;
        NoSeriesBatchLog.Insert(true);
    end;

    local procedure GetNextBatchLogEntryNo(): Integer
    var
        NoSeriesBatchLog: Record "No. Series Batch Log";
    begin
        if NoSeriesBatchLog.FindLast() then
            exit(NoSeriesBatchLog."Entry No." + 1)
        else
            exit(1);
    end;
}
```

## Multi-Dimensional Number Generation

```al
// Support for complex numbering schemes with multiple dimensions
codeunit 50503 "Multi-Dimensional No. Series"
{
    procedure GetNextCompoundNo(BaseSeriesCode: Code[20]; Dimensions: Dictionary of [Text, Text]): Code[20]
    var
        CompoundCode: Code[20];
        GeneratedNo: Code[20];
    begin
        CompoundCode := BuildCompoundSeriesCode(BaseSeriesCode, Dimensions);
        GeneratedNo := GetOrCreateCompoundSeries(CompoundCode, Dimensions);

        LogCompoundNumberGeneration(BaseSeriesCode, CompoundCode, GeneratedNo, Dimensions);

        exit(GeneratedNo);
    end;

    procedure GetLocationBasedNo(BaseSeriesCode: Code[20]; LocationCode: Code[10]): Code[20]
    var
        Dimensions: Dictionary of [Text, Text];
    begin
        Dimensions.Set('LOCATION', LocationCode);
        exit(GetNextCompoundNo(BaseSeriesCode, Dimensions));
    end;

    procedure GetYearBasedNo(BaseSeriesCode: Code[20]; FiscalYear: Integer): Code[20]
    var
        Dimensions: Dictionary of [Text, Text];
    begin
        Dimensions.Set('YEAR', Format(FiscalYear));
        exit(GetNextCompoundNo(BaseSeriesCode, Dimensions));
    end;

    procedure GetDepartmentBasedNo(BaseSeriesCode: Code[20]; DepartmentCode: Code[10]; LocationCode: Code[10]): Code[20]
    var
        Dimensions: Dictionary of [Text, Text];
    begin
        Dimensions.Set('DEPARTMENT', DepartmentCode);
        Dimensions.Set('LOCATION', LocationCode);
        exit(GetNextCompoundNo(BaseSeriesCode, Dimensions));
    end;

    local procedure BuildCompoundSeriesCode(BaseSeriesCode: Code[20]; Dimensions: Dictionary of [Text, Text]): Code[20]
    var
        CompoundCode: Text;
        DimensionKey: Text;
        DimensionValue: Text;
        SortedKeys: List of [Text];
    begin
        CompoundCode := BaseSeriesCode;

        // Sort dimension keys for consistent ordering
        foreach DimensionKey in Dimensions.Keys do
            SortedKeys.Add(DimensionKey);

        SortDimensionKeys(SortedKeys);

        // Build compound code
        foreach DimensionKey in SortedKeys do begin
            Dimensions.Get(DimensionKey, DimensionValue);
            CompoundCode += '-' + DimensionKey + '-' + DimensionValue;
        end;

        // Ensure code fits in 20 characters
        if StrLen(CompoundCode) > 20 then
            CompoundCode := CreateHashedCompoundCode(BaseSeriesCode, Dimensions);

        exit(CopyStr(CompoundCode, 1, 20));
    end;

    local procedure GetOrCreateCompoundSeries(CompoundCode: Code[20]; Dimensions: Dictionary of [Text, Text]): Code[20]
    var
        NoSeries: Record "No. Series";
        CustomNoSeriesManagement: Codeunit "Custom No. Series Management";
        StartingNo: Code[20];
        EndingNo: Code[20];
    begin
        if not NoSeries.Get(CompoundCode) then begin
            // Create new compound series
            StartingNo := BuildStartingNumber(Dimensions);
            EndingNo := BuildEndingNumber(Dimensions);

            if not CustomNoSeriesManagement.CreateCustomNoSeries(
                CompoundCode,
                'Auto-generated compound series',
                StartingNo,
                EndingNo) then
                Error('Failed to create compound No. Series %1', CompoundCode);
        end;

        exit(CustomNoSeriesManagement.GetNextNo(CompoundCode, Today));
    end;

    local procedure BuildStartingNumber(Dimensions: Dictionary of [Text, Text]): Code[20]
    var
        StartingNo: Text;
        DimensionKey: Text;
        DimensionValue: Text;
    begin
        StartingNo := '';

        // Build starting number based on dimensions
        if Dimensions.Get('LOCATION', DimensionValue) then
            StartingNo += DimensionValue;

        if Dimensions.Get('DEPARTMENT', DimensionValue) then
            StartingNo += DimensionValue;

        if Dimensions.Get('YEAR', DimensionValue) then
            StartingNo += CopyStr(DimensionValue, 3, 2); // Last 2 digits of year

        StartingNo += '00001';

        exit(CopyStr(StartingNo, 1, 20));
    end;

    local procedure BuildEndingNumber(Dimensions: Dictionary of [Text, Text]): Code[20]
    var
        EndingNo: Text;
        DimensionKey: Text;
        DimensionValue: Text;
    begin
        EndingNo := '';

        // Build ending number based on dimensions
        if Dimensions.Get('LOCATION', DimensionValue) then
            EndingNo += DimensionValue;

        if Dimensions.Get('DEPARTMENT', DimensionValue) then
            EndingNo += DimensionValue;

        if Dimensions.Get('YEAR', DimensionValue) then
            EndingNo += CopyStr(DimensionValue, 3, 2); // Last 2 digits of year

        EndingNo += '99999';

        exit(CopyStr(EndingNo, 1, 20));
    end;

    local procedure CreateHashedCompoundCode(BaseSeriesCode: Code[20]; Dimensions: Dictionary of [Text, Text]): Code[20]
    var
        HashInput: Text;
        DimensionKey: Text;
        DimensionValue: Text;
        HashValue: Text;
    begin
        HashInput := BaseSeriesCode;

        foreach DimensionKey in Dimensions.Keys do begin
            Dimensions.Get(DimensionKey, DimensionValue);
            HashInput += DimensionKey + DimensionValue;
        end;

        // Create simple hash (in production, use proper hashing)
        HashValue := Format(HashInput.GetHashCode(), 0, '<Integer,8><Filler Character,0>');

        exit(CopyStr(BaseSeriesCode + '-' + HashValue, 1, 20));
    end;

    local procedure SortDimensionKeys(var Keys: List of [Text])
    var
        i, j: Integer;
        temp: Text;
    begin
        // Simple bubble sort for demonstration
        for i := 1 to Keys.Count - 1 do begin
            for j := 1 to Keys.Count - i do begin
                if Keys.Get(j) > Keys.Get(j + 1) then begin
                    temp := Keys.Get(j);
                    Keys.Set(j, Keys.Get(j + 1));
                    Keys.Set(j + 1, temp);
                end;
            end;
        end;
    end;

    local procedure LogCompoundNumberGeneration(BaseSeriesCode: Code[20]; CompoundCode: Code[20]; GeneratedNo: Code[20]; Dimensions: Dictionary of [Text, Text])
    var
        CompoundNoLog: Record "Compound No. Log";
        DimensionText: Text;
        DimensionKey: Text;
        DimensionValue: Text;
    begin
        // Build dimension text
        foreach DimensionKey in Dimensions.Keys do begin
            Dimensions.Get(DimensionKey, DimensionValue);
            if DimensionText <> '' then
                DimensionText += '; ';
            DimensionText += DimensionKey + '=' + DimensionValue;
        end;

        CompoundNoLog.Init();
        CompoundNoLog."Entry No." := GetNextCompoundLogEntryNo();
        CompoundNoLog."Base Series Code" := BaseSeriesCode;
        CompoundNoLog."Compound Series Code" := CompoundCode;
        CompoundNoLog."Generated No." := GeneratedNo;
        CompoundNoLog."Dimensions" := CopyStr(DimensionText, 1, 250);
        CompoundNoLog."Generation Date" := Today;
        CompoundNoLog."Generation Time" := Time;
        CompoundNoLog."User ID" := UserId;
        CompoundNoLog.Insert(true);
    end;

    local procedure GetNextCompoundLogEntryNo(): Integer
    var
        CompoundNoLog: Record "Compound No. Log";
    begin
        if CompoundNoLog.FindLast() then
            exit(CompoundNoLog."Entry No." + 1)
        else
            exit(1);
    end;
}
```

This comprehensive implementation provides robust number generation capabilities including basic number series management, concurrency control, batch allocation, and multi-dimensional numbering for complex Business Central scenarios.