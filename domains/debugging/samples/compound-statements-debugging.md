# AL Compound Statements for Enhanced Debugging Examples

## Strategic Breakpoint Placement

### Clear Breakpoint Locations with Compound Statements
```al
local procedure ProcessCustomerOrder(var SalesHeader: Record "Sales Header"): Boolean
var
    Customer: Record Customer;
    ValidationResult: Boolean;
begin
    // Breakpoint here provides clear entry context
    if Customer.Get(SalesHeader."Sell-to Customer No.") then begin
        // Breakpoint here confirms customer retrieval success
        ValidationResult := ValidateCustomerData(Customer);
        
        if ValidationResult then begin
            // Breakpoint here enters successful validation path
            if ProcessOrderValidation(SalesHeader) then begin
                // Breakpoint here confirms order validation success
                ProcessApprovedOrder(SalesHeader);
                exit(true);
            end else begin
                // Breakpoint here enters order validation failure path
                LogOrderValidationFailure(SalesHeader."No.");
                exit(false);
            end;
        end else begin
            // Breakpoint here enters customer validation failure path
            LogCustomerValidationFailure(Customer."No.");
            exit(false);
        end;
    end else begin
        // Breakpoint here captures customer not found scenario
        LogCustomerNotFound(SalesHeader."Sell-to Customer No.");
        exit(false);
    end;
end;
```

## Exception Context Enhancement

### Clear Exception Boundaries with Compound Statements
```al
local procedure ProcessDocumentPosting(var GenJournalLine: Record "Gen. Journal Line")
var
    PostingResult: Boolean;
begin
    try
        // Compound block provides clear exception scope
        if ValidateJournalLine(GenJournalLine) then begin
            if CheckAccountingPeriod(GenJournalLine."Posting Date") then begin
                PostingResult := PostJournalLine(GenJournalLine);
                
                if PostingResult then begin
                    // Breakpoint here confirms successful posting
                    LogSuccessfulPosting(GenJournalLine."Document No.");
                end else begin
                    // Breakpoint here captures posting failure
                    LogPostingFailure(GenJournalLine."Document No.");
                end;
            end else begin
                // Breakpoint here captures period validation failure
                Error('Accounting period is closed for date %1', GenJournalLine."Posting Date");
            end;
        end else begin
            // Breakpoint here captures journal line validation failure
            Error('Journal line validation failed');
        end;
    except
        on E: Exception do begin
            // Breakpoint here provides clear exception context
            LogPostingException(GenJournalLine."Document No.", E.Message);
            
            if E.Message.Contains('period') then begin
                // Breakpoint here handles period-specific exceptions
                HandlePeriodException(GenJournalLine);
            end else begin
                // Breakpoint here handles general posting exceptions
                HandleGeneralPostingException(GenJournalLine, E);
            end;
        end;
    end;
end;
```

## Loop Debugging with Compound Statements

### Clear Loop Iteration Boundaries
```al
local procedure ProcessItemBatch(var ItemBuffer: Record "Item Buffer")
var
    ProcessedCount: Integer;
    ErrorCount: Integer;
    CurrentItem: Code[20];
begin
    ProcessedCount := 0;
    ErrorCount := 0;
    
    if ItemBuffer.FindSet() then begin
        // Breakpoint here confirms record set found
        repeat
            CurrentItem := ItemBuffer."No.";
            
            // Breakpoint here shows current item being processed
            if ValidateItemData(ItemBuffer) then begin
                // Breakpoint here enters successful validation path
                try
                    if UpdateItemCosts(ItemBuffer) then begin
                        // Breakpoint here confirms successful cost update
                        ProcessedCount += 1;
                        LogItemProcessed(CurrentItem);
                    end else begin
                        // Breakpoint here captures cost update failure
                        ErrorCount += 1;
                        LogItemError(CurrentItem, 'Cost update failed');
                    end;
                except
                    on E: Exception do begin
                        // Breakpoint here captures processing exceptions
                        ErrorCount += 1;
                        LogItemException(CurrentItem, E.Message);
                    end;
                end;
            end else begin
                // Breakpoint here captures validation failures
                ErrorCount += 1;
                LogItemValidationError(CurrentItem);
            end;
        until ItemBuffer.Next() = 0;
        
        // Breakpoint here provides final processing summary
        LogBatchSummary(ProcessedCount, ErrorCount);
    end else begin
        // Breakpoint here captures empty batch scenario
        LogEmptyBatch();
    end;
end;
```