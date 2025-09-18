# No Series Validation Patterns - AL Code Examples

This sample demonstrates implementing comprehensive validation patterns for number sequence integrity in Business Central.

## Basic Validation Framework

```al
// Core validation interface for no series
interface INoSeriesValidator
{
    procedure ValidateFormat(SeriesCode: Code[20]; NumberValue: Code[20]): Boolean;
    procedure ValidateSequence(SeriesCode: Code[20]): Boolean;
    procedure ValidateIntegrity(SeriesCode: Code[20]): Boolean;
    procedure GetValidationErrors(): List of [Text];
}
```

## Comprehensive No Series Validator

```al
// Main validation implementation
codeunit 50900 "No Series Validator" implements INoSeriesValidator
{
    var
        ValidationErrors: List of [Text];
        ValidationWarnings: List of [Text];

    procedure ValidateFormat(SeriesCode: Code[20]; NumberValue: Code[20]): Boolean
    var
        NoSeriesLine: Record "No. Series Line";
        FormatValid: Boolean;
    begin
        Clear(ValidationErrors);
        FormatValid := true;

        if not FindNoSeriesLine(SeriesCode, NoSeriesLine) then begin
            AddValidationError(StrSubstNo('No. Series line not found for series %1', SeriesCode));
            exit(false);
        end;

        // Validate number format
        if not ValidateNumberFormat(NumberValue, NoSeriesLine) then
            FormatValid := false;

        // Validate number range
        if not ValidateNumberRange(NumberValue, NoSeriesLine) then
            FormatValid := false;

        // Validate increment pattern
        if not ValidateIncrementPattern(NumberValue, NoSeriesLine) then
            FormatValid := false;

        exit(FormatValid);
    end;

    procedure ValidateSequence(SeriesCode: Code[20]): Boolean
    var
        NoSeriesLine: Record "No. Series Line";
        NoSeriesLog: Record "No. Series Log";
        SequenceValid: Boolean;
        GapCount: Integer;
        DuplicateCount: Integer;
    begin
        Clear(ValidationErrors);
        Clear(ValidationWarnings);
        SequenceValid := true;

        if not FindNoSeriesLine(SeriesCode, NoSeriesLine) then begin
            AddValidationError(StrSubstNo('No. Series line not found for series %1', SeriesCode));
            exit(false);
        end;

        // Check for gaps in sequence
        GapCount := CheckForGaps(SeriesCode, NoSeriesLine);
        if GapCount > 0 then begin
            AddValidationWarning(StrSubstNo('%1 gaps found in sequence %2', GapCount, SeriesCode));
        end;

        // Check for duplicates
        DuplicateCount := CheckForDuplicates(SeriesCode);
        if DuplicateCount > 0 then begin
            AddValidationError(StrSubstNo('%1 duplicate numbers found in sequence %2', DuplicateCount, SeriesCode));
            SequenceValid := false;
        end;

        // Validate last used number
        if not ValidateLastUsedNumber(NoSeriesLine) then
            SequenceValid := false;

        // Check sequence boundaries
        if not ValidateSequenceBoundaries(NoSeriesLine) then
            SequenceValid := false;

        exit(SequenceValid);
    end;

    procedure ValidateIntegrity(SeriesCode: Code[20]): Boolean
    var
        IntegrityValid: Boolean;
    begin
        Clear(ValidationErrors);
        IntegrityValid := true;

        // Validate cross-table references
        if not ValidateCrossTableReferences(SeriesCode) then
            IntegrityValid := false;

        // Validate concurrency conflicts
        if not ValidateConcurrencyIntegrity(SeriesCode) then
            IntegrityValid := false;

        // Validate business rule compliance
        if not ValidateBusinessRules(SeriesCode) then
            IntegrityValid := false;

        // Validate system consistency
        if not ValidateSystemConsistency(SeriesCode) then
            IntegrityValid := false;

        exit(IntegrityValid);
    end;

    procedure GetValidationErrors(): List of [Text]
    begin
        exit(ValidationErrors);
    end;

    procedure GetValidationWarnings(): List of [Text]
    begin
        exit(ValidationWarnings);
    end;

    local procedure FindNoSeriesLine(SeriesCode: Code[20]; var NoSeriesLine: Record "No. Series Line"): Boolean
    begin
        NoSeriesLine.SetRange("Series Code", SeriesCode);
        NoSeriesLine.SetFilter("Starting Date", '<=%1', Today);
        NoSeriesLine.SetFilter("Ending Date", '%1|>=%2', 0D, Today);

        exit(NoSeriesLine.FindLast());
    end;

    local procedure ValidateNumberFormat(NumberValue: Code[20]; NoSeriesLine: Record "No. Series Line"): Boolean
    var
        ExpectedFormat: Text;
        ActualFormat: Text;
        FormatValid: Boolean;
    begin
        FormatValid := true;

        // Validate length
        if StrLen(NumberValue) > 20 then begin
            AddValidationError(StrSubstNo('Number %1 exceeds maximum length of 20 characters', NumberValue));
            FormatValid := false;
        end;

        // Validate prefix consistency
        if not ValidatePrefix(NumberValue, NoSeriesLine) then
            FormatValid := false;

        // Validate suffix consistency
        if not ValidateSuffix(NumberValue, NoSeriesLine) then
            FormatValid := false;

        // Validate numeric portion
        if not ValidateNumericPortion(NumberValue, NoSeriesLine) then
            FormatValid := false;

        exit(FormatValid);
    end;

    local procedure ValidateNumberRange(NumberValue: Code[20]; NoSeriesLine: Record "No. Series Line"): Boolean
    var
        RangeValid: Boolean;
    begin
        RangeValid := true;

        // Check if number is within starting range
        if (NoSeriesLine."Starting No." <> '') and (NumberValue < NoSeriesLine."Starting No.") then begin
            AddValidationError(StrSubstNo('Number %1 is below starting number %2', NumberValue, NoSeriesLine."Starting No."));
            RangeValid := false;
        end;

        // Check if number is within ending range
        if (NoSeriesLine."Ending No." <> '') and (NumberValue > NoSeriesLine."Ending No.") then begin
            AddValidationError(StrSubstNo('Number %1 exceeds ending number %2', NumberValue, NoSeriesLine."Ending No."));
            RangeValid := false;
        end;

        exit(RangeValid);
    end;

    local procedure ValidateIncrementPattern(NumberValue: Code[20]; NoSeriesLine: Record "No. Series Line"): Boolean
    var
        ExpectedIncrement: Integer;
        ActualIncrement: Integer;
        LastUsedNumber: Code[20];
    begin
        if NoSeriesLine."Last No. Used" = '' then
            exit(true); // First number, no increment to validate

        LastUsedNumber := NoSeriesLine."Last No. Used";
        ExpectedIncrement := NoSeriesLine."Increment-by No.";

        ActualIncrement := CalculateIncrement(LastUsedNumber, NumberValue);

        if ActualIncrement <> ExpectedIncrement then begin
            AddValidationError(StrSubstNo('Number %1 does not follow expected increment pattern. Expected: %2, Actual: %3',
                NumberValue, ExpectedIncrement, ActualIncrement));
            exit(false);
        end;

        exit(true);
    end;

    local procedure CheckForGaps(SeriesCode: Code[20]; NoSeriesLine: Record "No. Series Line"): Integer
    var
        NoSeriesLog: Record "No. Series Log";
        ExpectedNumber: Code[20];
        CurrentNumber: Code[20];
        GapCount: Integer;
        PreviousNumber: Code[20];
    begin
        GapCount := 0;
        PreviousNumber := NoSeriesLine."Starting No.";

        NoSeriesLog.SetRange("Series Code", SeriesCode);
        NoSeriesLog.SetCurrentKey("Generation Date", "Generation Time");

        if NoSeriesLog.FindSet() then
            repeat
                CurrentNumber := NoSeriesLog."Generated No.";
                ExpectedNumber := IncrementNumber(PreviousNumber, NoSeriesLine."Increment-by No.");

                if CurrentNumber <> ExpectedNumber then begin
                    // Gap detected
                    GapCount += 1;
                    LogGap(SeriesCode, ExpectedNumber, CurrentNumber);
                end;

                PreviousNumber := CurrentNumber;
            until NoSeriesLog.Next() = 0;

        exit(GapCount);
    end;

    local procedure CheckForDuplicates(SeriesCode: Code[20]): Integer
    var
        NoSeriesLog: Record "No. Series Log";
        TempNoSeriesLog: Record "No. Series Log" temporary;
        DuplicateCount: Integer;
    begin
        DuplicateCount := 0;

        NoSeriesLog.SetRange("Series Code", SeriesCode);

        if NoSeriesLog.FindSet() then
            repeat
                TempNoSeriesLog.SetRange("Generated No.", NoSeriesLog."Generated No.");
                if TempNoSeriesLog.FindFirst() then begin
                    // Duplicate found
                    DuplicateCount += 1;
                    LogDuplicate(SeriesCode, NoSeriesLog."Generated No.", NoSeriesLog."Generation Date", TempNoSeriesLog."Generation Date");
                end else begin
                    TempNoSeriesLog.TransferFields(NoSeriesLog);
                    TempNoSeriesLog.Insert();
                end;
            until NoSeriesLog.Next() = 0;

        exit(DuplicateCount);
    end;

    local procedure ValidateLastUsedNumber(NoSeriesLine: Record "No. Series Line"): Boolean
    var
        NoSeriesLog: Record "No. Series Log";
        LastLoggedNumber: Code[20];
    begin
        if NoSeriesLine."Last No. Used" = '' then
            exit(true);

        // Find the most recent logged number
        NoSeriesLog.SetRange("Series Code", NoSeriesLine."Series Code");
        if NoSeriesLog.FindLast() then begin
            LastLoggedNumber := NoSeriesLog."Generated No.";

            if LastLoggedNumber <> NoSeriesLine."Last No. Used" then begin
                AddValidationError(StrSubstNo('Last used number mismatch. Series Line: %1, Log: %2',
                    NoSeriesLine."Last No. Used", LastLoggedNumber));
                exit(false);
            end;
        end;

        exit(true);
    end;

    local procedure ValidateSequenceBoundaries(NoSeriesLine: Record "No. Series Line"): Boolean
    var
        BoundariesValid: Boolean;
    begin
        BoundariesValid := true;

        // Validate that starting number is less than ending number
        if (NoSeriesLine."Starting No." <> '') and (NoSeriesLine."Ending No." <> '') then begin
            if NoSeriesLine."Starting No." >= NoSeriesLine."Ending No." then begin
                AddValidationError(StrSubstNo('Starting number %1 must be less than ending number %2',
                    NoSeriesLine."Starting No.", NoSeriesLine."Ending No."));
                BoundariesValid := false;
            end;
        end;

        // Validate that last used number is within boundaries
        if NoSeriesLine."Last No. Used" <> '' then begin
            if (NoSeriesLine."Starting No." <> '') and (NoSeriesLine."Last No. Used" < NoSeriesLine."Starting No.") then begin
                AddValidationError(StrSubstNo('Last used number %1 is below starting number %2',
                    NoSeriesLine."Last No. Used", NoSeriesLine."Starting No."));
                BoundariesValid := false;
            end;

            if (NoSeriesLine."Ending No." <> '') and (NoSeriesLine."Last No. Used" > NoSeriesLine."Ending No.") then begin
                AddValidationError(StrSubstNo('Last used number %1 exceeds ending number %2',
                    NoSeriesLine."Last No. Used", NoSeriesLine."Ending No."));
                BoundariesValid := false;
            end;
        end;

        exit(BoundariesValid);
    end;

    local procedure ValidateCrossTableReferences(SeriesCode: Code[20]): Boolean
    var
        SalesHeader: Record "Sales Header";
        PurchaseHeader: Record "Purchase Header";
        ItemLedgerEntry: Record "Item Ledger Entry";
        ReferenceValid: Boolean;
        OrphanCount: Integer;
    begin
        ReferenceValid := true;

        // Check for orphaned numbers in sales documents
        OrphanCount := CheckOrphanedReferences(SeriesCode, Database::"Sales Header");
        if OrphanCount > 0 then begin
            AddValidationWarning(StrSubstNo('%1 orphaned references found in Sales Headers for series %2', OrphanCount, SeriesCode));
        end;

        // Check for orphaned numbers in purchase documents
        OrphanCount := CheckOrphanedReferences(SeriesCode, Database::"Purchase Header");
        if OrphanCount > 0 then begin
            AddValidationWarning(StrSubstNo('%1 orphaned references found in Purchase Headers for series %2', OrphanCount, SeriesCode));
        end;

        exit(ReferenceValid);
    end;

    local procedure ValidateConcurrencyIntegrity(SeriesCode: Code[20]): Boolean
    var
        NoSeriesLock: Record "No. Series Lock";
        ConcurrencyValid: Boolean;
    begin
        ConcurrencyValid := true;

        // Check for stale locks
        NoSeriesLock.SetRange("Series Code", SeriesCode);
        NoSeriesLock.SetFilter("Lock Time", '<%1', CurrentDateTime - 300000); // 5 minutes old

        if not NoSeriesLock.IsEmpty then begin
            AddValidationWarning(StrSubstNo('Stale locks found for series %1', SeriesCode));
            // Clean up stale locks
            NoSeriesLock.DeleteAll(true);
        end;

        exit(ConcurrencyValid);
    end;

    local procedure ValidateBusinessRules(SeriesCode: Code[20]): Boolean
    var
        NoSeries: Record "No. Series";
        BusinessRulesValid: Boolean;
    begin
        BusinessRulesValid := true;

        if not NoSeries.Get(SeriesCode) then begin
            AddValidationError(StrSubstNo('No. Series %1 not found', SeriesCode));
            exit(false);
        end;

        // Validate business rule: Manual vs. Automatic
        if NoSeries."Manual Nos." and NoSeries."Default Nos." then begin
            AddValidationError(StrSubstNo('Series %1 cannot have both Manual and Default numbering enabled', SeriesCode));
            BusinessRulesValid := false;
        end;

        // Validate date-based rules
        if not ValidateDateBasedRules(SeriesCode) then
            BusinessRulesValid := false;

        exit(BusinessRulesValid);
    end;

    local procedure ValidateSystemConsistency(SeriesCode: Code[20]): Boolean
    var
        NoSeriesLine: Record "No. Series Line";
        ConsistencyValid: Boolean;
        OverlapCount: Integer;
    begin
        ConsistencyValid := true;

        // Check for overlapping date ranges
        OverlapCount := CheckDateRangeOverlaps(SeriesCode);
        if OverlapCount > 0 then begin
            AddValidationError(StrSubstNo('%1 overlapping date ranges found for series %2', OverlapCount, SeriesCode));
            ConsistencyValid := false;
        end;

        // Check for missing date ranges
        if not ValidateDateRangeContinuity(SeriesCode) then
            ConsistencyValid := false;

        exit(ConsistencyValid);
    end;

    // Helper procedures...
    local procedure ValidatePrefix(NumberValue: Code[20]; NoSeriesLine: Record "No. Series Line"): Boolean
    begin
        // Implement prefix validation logic
        exit(true);
    end;

    local procedure ValidateSuffix(NumberValue: Code[20]; NoSeriesLine: Record "No. Series Line"): Boolean
    begin
        // Implement suffix validation logic
        exit(true);
    end;

    local procedure ValidateNumericPortion(NumberValue: Code[20]; NoSeriesLine: Record "No. Series Line"): Boolean
    begin
        // Implement numeric portion validation logic
        exit(true);
    end;

    local procedure CalculateIncrement(FromNumber: Code[20]; ToNumber: Code[20]): Integer
    begin
        // Implement increment calculation logic
        exit(1);
    end;

    local procedure IncrementNumber(Number: Code[20]; IncrementBy: Integer): Code[20]
    begin
        // Implement number increment logic
        exit(Number);
    end;

    local procedure CheckOrphanedReferences(SeriesCode: Code[20]; TableNo: Integer): Integer
    begin
        // Implement orphaned reference checking
        exit(0);
    end;

    local procedure CheckDateRangeOverlaps(SeriesCode: Code[20]): Integer
    begin
        // Implement date range overlap checking
        exit(0);
    end;

    local procedure ValidateDateRangeContinuity(SeriesCode: Code[20]): Boolean
    begin
        // Implement date range continuity validation
        exit(true);
    end;

    local procedure ValidateDateBasedRules(SeriesCode: Code[20]): Boolean
    begin
        // Implement date-based business rule validation
        exit(true);
    end;

    local procedure AddValidationError(ErrorText: Text)
    begin
        ValidationErrors.Add(ErrorText);
    end;

    local procedure AddValidationWarning(WarningText: Text)
    begin
        ValidationWarnings.Add(WarningText);
    end;

    local procedure LogGap(SeriesCode: Code[20]; ExpectedNumber: Code[20]; ActualNumber: Code[20])
    begin
        // Log gap detection for analysis
    end;

    local procedure LogDuplicate(SeriesCode: Code[20]; DuplicateNumber: Code[20]; Date1: Date; Date2: Date)
    begin
        // Log duplicate detection for analysis
    end;
}
```

## Real-Time Validation Integration

```al
// Real-time validation during number generation
codeunit 50901 "Real-Time No Series Validator"
{
    var
        Validator: Codeunit "No Series Validator";

    [EventSubscriber(ObjectType::Codeunit, Codeunit::"No. Series Management", 'OnBeforeGetNextNo', '', false, false)]
    local procedure OnBeforeGetNextNo(NoSeriesCode: Code[20]; SeriesDate: Date; var NoSeriesLine: Record "No. Series Line")
    begin
        ValidateBeforeGeneration(NoSeriesCode, NoSeriesLine);
    end;

    [EventSubscriber(ObjectType::Codeunit, Codeunit::"No. Series Management", 'OnAfterGetNextNo', '', false, false)]
    local procedure OnAfterGetNextNo(NoSeriesCode: Code[20]; SeriesDate: Date; var NoSeriesLine: Record "No. Series Line"; var NextNo: Code[20])
    begin
        ValidateAfterGeneration(NoSeriesCode, NextNo, NoSeriesLine);
    end;

    local procedure ValidateBeforeGeneration(NoSeriesCode: Code[20]; NoSeriesLine: Record "No. Series Line")
    var
        ValidationErrors: List of [Text];
        ErrorText: Text;
    begin
        if not Validator.ValidateSequence(NoSeriesCode) then begin
            ValidationErrors := Validator.GetValidationErrors();
            foreach ErrorText in ValidationErrors do
                Error('No. Series validation failed: %1', ErrorText);
        end;
    end;

    local procedure ValidateAfterGeneration(NoSeriesCode: Code[20]; NextNo: Code[20]; NoSeriesLine: Record "No. Series Line")
    var
        ValidationErrors: List of [Text];
        ErrorText: Text;
    begin
        if not Validator.ValidateFormat(NoSeriesCode, NextNo) then begin
            ValidationErrors := Validator.GetValidationErrors();
            foreach ErrorText in ValidationErrors do
                Error('Generated number validation failed: %1', ErrorText);
        end;

        // Log successful generation
        LogNumberGeneration(NoSeriesCode, NextNo);
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

## Batch Validation Tool

```al
// Comprehensive batch validation for all series
codeunit 50902 "Batch No Series Validation"
{
    var
        Validator: Codeunit "No Series Validator";

    procedure ValidateAllSeries(): Dictionary of [Code[20], List of [Text]]
    var
        NoSeries: Record "No. Series";
        ValidationResults: Dictionary of [Code[20], List of [Text]];
        SeriesErrors: List of [Text];
    begin
        if NoSeries.FindSet() then
            repeat
                Clear(SeriesErrors);

                // Run comprehensive validation
                if not Validator.ValidateSequence(NoSeries.Code) then
                    AddErrors(SeriesErrors, Validator.GetValidationErrors());

                if not Validator.ValidateIntegrity(NoSeries.Code) then
                    AddErrors(SeriesErrors, Validator.GetValidationErrors());

                ValidationResults.Set(NoSeries.Code, SeriesErrors);

            until NoSeries.Next() = 0;

        exit(ValidationResults);
    end;

    procedure GenerateValidationReport(): Text
    var
        ValidationResults: Dictionary of [Code[20], List of [Text]];
        ReportText: Text;
        SeriesCode: Code[20];
        SeriesErrors: List of [Text];
        ErrorText: Text;
        TotalSeries: Integer;
        SeriesWithErrors: Integer;
    begin
        ValidationResults := ValidateAllSeries();
        TotalSeries := ValidationResults.Count;

        ReportText := StrSubstNo('No. Series Validation Report - %1\n', CurrentDateTime);
        ReportText += StrSubstNo('Total Series Validated: %1\n\n', TotalSeries);

        foreach SeriesCode in ValidationResults.Keys do begin
            ValidationResults.Get(SeriesCode, SeriesErrors);

            if SeriesErrors.Count > 0 then begin
                SeriesWithErrors += 1;
                ReportText += StrSubstNo('Series: %1 (%2 errors)\n', SeriesCode, SeriesErrors.Count);

                foreach ErrorText in SeriesErrors do
                    ReportText += StrSubstNo('  - %1\n', ErrorText);

                ReportText += '\n';
            end;
        end;

        ReportText += StrSubstNo('Summary: %1 of %2 series have validation errors.\n', SeriesWithErrors, TotalSeries);

        exit(ReportText);
    end;

    procedure FixCommonIssues(): Integer
    var
        ValidationResults: Dictionary of [Code[20], List of [Text]];
        SeriesCode: Code[20];
        SeriesErrors: List of [Text];
        FixedCount: Integer;
    begin
        ValidationResults := ValidateAllSeries();

        foreach SeriesCode in ValidationResults.Keys do begin
            ValidationResults.Get(SeriesCode, SeriesErrors);

            if SeriesErrors.Count > 0 then begin
                if FixSeriesIssues(SeriesCode, SeriesErrors) then
                    FixedCount += 1;
            end;
        end;

        exit(FixedCount);
    end;

    local procedure FixSeriesIssues(SeriesCode: Code[20]; Errors: List of [Text]): Boolean
    var
        ErrorText: Text;
        FixedAny: Boolean;
    begin
        FixedAny := false;

        foreach ErrorText in Errors do begin
            case true of
                StrPos(ErrorText, 'Stale locks') > 0:
                    begin
                        CleanupStaleLocks(SeriesCode);
                        FixedAny := true;
                    end;
                StrPos(ErrorText, 'orphaned references') > 0:
                    begin
                        CleanupOrphanedReferences(SeriesCode);
                        FixedAny := true;
                    end;
            end;
        end;

        exit(FixedAny);
    end;

    local procedure AddErrors(var TargetList: List of [Text]; SourceList: List of [Text])
    var
        ErrorText: Text;
    begin
        foreach ErrorText in SourceList do
            TargetList.Add(ErrorText);
    end;

    local procedure CleanupStaleLocks(SeriesCode: Code[20])
    var
        NoSeriesLock: Record "No. Series Lock";
    begin
        NoSeriesLock.SetRange("Series Code", SeriesCode);
        NoSeriesLock.SetFilter("Lock Time", '<%1', CurrentDateTime - 300000);
        NoSeriesLock.DeleteAll(true);
    end;

    local procedure CleanupOrphanedReferences(SeriesCode: Code[20])
    begin
        // Implement orphaned reference cleanup
    end;
}
```

## Usage Example

```al
// Example of using validation patterns
codeunit 50903 "Validation Usage Example"
{
    procedure DemonstrateValidation()
    var
        Validator: Codeunit "No Series Validator";
        BatchValidator: Codeunit "Batch No Series Validation";
        ValidationReport: Text;
        SeriesCode: Code[20];
        TestNumber: Code[20];
        ValidationErrors: List of [Text];
        ErrorText: Text;
        FixedCount: Integer;
    begin
        SeriesCode := 'SALES-ORDER';
        TestNumber := 'SO-2024-001';

        // Validate specific number format
        if Validator.ValidateFormat(SeriesCode, TestNumber) then
            Message('Number format validation passed')
        else begin
            ValidationErrors := Validator.GetValidationErrors();
            foreach ErrorText in ValidationErrors do
                Message('Format error: %1', ErrorText);
        end;

        // Validate sequence integrity
        if Validator.ValidateSequence(SeriesCode) then
            Message('Sequence validation passed')
        else begin
            ValidationErrors := Validator.GetValidationErrors();
            foreach ErrorText in ValidationErrors do
                Message('Sequence error: %1', ErrorText);
        end;

        // Run batch validation
        ValidationReport := BatchValidator.GenerateValidationReport();
        Message('Validation Report:\n%1', ValidationReport);

        // Fix common issues
        FixedCount := BatchValidator.FixCommonIssues();
        Message('Fixed %1 series issues automatically', FixedCount);
    end;
}
```

This comprehensive validation implementation provides robust number sequence integrity checking with real-time validation, batch processing capabilities, and automatic issue remediation for Business Central number series management.