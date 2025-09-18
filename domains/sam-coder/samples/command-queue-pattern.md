# Command Queue Pattern in AL - Code Examples

This sample demonstrates implementing command queue patterns for asynchronous processing in Business Central.

## Basic Command Interface

```al
// Core command interface
interface ICommand
{
    procedure Execute(): Boolean;
    procedure GetCommandName(): Text[50];
    procedure GetDescription(): Text[250];
    procedure CanUndo(): Boolean;
    procedure Undo(): Boolean;
    procedure GetExecutionTime(): DateTime;
    procedure GetCommandData(): Dictionary of [Text, Variant];
}
```

## Command Queue Implementation

```al
// Main command queue system
codeunit 50700 "Command Queue Manager"
{
    var
        CommandQueue: List of [Interface ICommand];
        ExecutedCommands: List of [Interface ICommand];
        IsProcessing: Boolean;
        ProcessingStats: Dictionary of [Text, Variant];

    procedure EnqueueCommand(Command: Interface ICommand)
    begin
        CommandQueue.Add(Command);
        LogCommandEnqueued(Command);

        // Auto-process if not already processing
        if not IsProcessing then
            ProcessNextCommand();
    end;

    procedure EnqueuePriorityCommand(Command: Interface ICommand)
    begin
        CommandQueue.Insert(1, Command);
        LogCommandEnqueued(Command);

        if not IsProcessing then
            ProcessNextCommand();
    end;

    procedure ProcessAllCommands(): Integer
    var
        ProcessedCount: Integer;
    begin
        ProcessedCount := 0;

        while CommandQueue.Count > 0 do begin
            if ProcessNextCommand() then
                ProcessedCount += 1
            else
                break; // Stop processing on error
        end;

        exit(ProcessedCount);
    end;

    procedure ProcessNextCommand(): Boolean
    var
        Command: Interface ICommand;
        ExecutionResult: Boolean;
        StartTime: DateTime;
        EndTime: DateTime;
    begin
        if CommandQueue.Count = 0 then
            exit(false);

        if IsProcessing then
            exit(false); // Prevent concurrent processing

        IsProcessing := true;
        Command := CommandQueue.Get(1);
        CommandQueue.RemoveAt(1);

        try
            StartTime := CurrentDateTime;
            ExecutionResult := ExecuteCommandSafely(Command);
            EndTime := CurrentDateTime;

            if ExecutionResult then begin
                ExecutedCommands.Add(Command);
                LogCommandExecuted(Command, StartTime, EndTime, true);
                UpdateProcessingStats(Command, true, EndTime - StartTime);
            end else begin
                LogCommandExecuted(Command, StartTime, EndTime, false);
                HandleCommandFailure(Command);
                UpdateProcessingStats(Command, false, EndTime - StartTime);
            end;

        finally
            IsProcessing := false;
        end;

        exit(ExecutionResult);
    end;

    procedure UndoLastCommand(): Boolean
    var
        LastCommand: Interface ICommand;
    begin
        if ExecutedCommands.Count = 0 then
            exit(false);

        LastCommand := ExecutedCommands.Get(ExecutedCommands.Count);

        if not LastCommand.CanUndo() then
            exit(false);

        if LastCommand.Undo() then begin
            ExecutedCommands.RemoveAt(ExecutedCommands.Count);
            LogCommandUndone(LastCommand);
            exit(true);
        end;

        exit(false);
    end;

    procedure GetQueueStatus(): Dictionary of [Text, Variant]
    var
        Status: Dictionary of [Text, Variant];
    begin
        Status.Set('QueueLength', CommandQueue.Count);
        Status.Set('ExecutedCount', ExecutedCommands.Count);
        Status.Set('IsProcessing', IsProcessing);
        Status.Set('ProcessingStats', ProcessingStats);

        exit(Status);
    end;

    procedure ClearQueue()
    begin
        CommandQueue.Clear();
        LogQueueCleared();
    end;

    local procedure ExecuteCommandSafely(Command: Interface ICommand): Boolean
    begin
        try
            exit(Command.Execute());
        except
            LogCommandError(Command, GetLastErrorText);
            exit(false);
        end;
    end;

    local procedure HandleCommandFailure(Command: Interface ICommand)
    begin
        // Add to dead letter queue or retry logic could go here
        LogCommandFailure(Command);
    end;

    local procedure UpdateProcessingStats(Command: Interface ICommand; Success: Boolean; Duration: BigInteger)
    var
        TotalCommands: Integer;
        SuccessfulCommands: Integer;
        FailedCommands: Integer;
        TotalDuration: BigInteger;
        AverageDuration: BigInteger;
    begin
        if ProcessingStats.Get('TotalCommands', TotalCommands) then
            TotalCommands += 1
        else
            TotalCommands := 1;

        if Success then begin
            if ProcessingStats.Get('SuccessfulCommands', SuccessfulCommands) then
                SuccessfulCommands += 1
            else
                SuccessfulCommands := 1;
        end else begin
            if ProcessingStats.Get('FailedCommands', FailedCommands) then
                FailedCommands += 1
            else
                FailedCommands := 1;
        end;

        if ProcessingStats.Get('TotalDuration', TotalDuration) then
            TotalDuration += Duration
        else
            TotalDuration := Duration;

        AverageDuration := TotalDuration div TotalCommands;

        ProcessingStats.Set('TotalCommands', TotalCommands);
        ProcessingStats.Set('SuccessfulCommands', SuccessfulCommands);
        ProcessingStats.Set('FailedCommands', FailedCommands);
        ProcessingStats.Set('TotalDuration', TotalDuration);
        ProcessingStats.Set('AverageDuration', AverageDuration);
    end;

    // Logging procedures
    local procedure LogCommandEnqueued(Command: Interface ICommand)
    begin
        // Log command enqueue event
    end;

    local procedure LogCommandExecuted(Command: Interface ICommand; StartTime: DateTime; EndTime: DateTime; Success: Boolean)
    begin
        // Log command execution details
    end;

    local procedure LogCommandUndone(Command: Interface ICommand)
    begin
        // Log command undo event
    end;

    local procedure LogCommandError(Command: Interface ICommand; ErrorText: Text)
    begin
        // Log command execution error
    end;

    local procedure LogCommandFailure(Command: Interface ICommand)
    begin
        // Log command failure for analysis
    end;

    local procedure LogQueueCleared()
    begin
        // Log queue clear event
    end;
}
```

## Sales Document Creation Command

```al
// Example command: Create sales document
codeunit 50701 "Create Sales Document Cmd" implements ICommand
{
    var
        CustomerNo: Code[20];
        Items: List of [Dictionary of [Text, Variant]];
        OrderDate: Date;
        CommandData: Dictionary of [Text, Variant];
        CreatedDocumentNo: Code[20];
        ExecutionTime: DateTime;

    procedure Initialize(NewCustomerNo: Code[20]; OrderItems: List of [Dictionary of [Text, Variant]]; NewOrderDate: Date)
    begin
        CustomerNo := NewCustomerNo;
        Items := OrderItems;
        OrderDate := NewOrderDate;

        // Store command data for logging/undo
        CommandData.Set('CustomerNo', CustomerNo);
        CommandData.Set('OrderDate', OrderDate);
        CommandData.Set('ItemCount', Items.Count);
    end;

    procedure Execute(): Boolean
    var
        SalesHeader: Record "Sales Header";
        SalesLine: Record "Sales Line";
        ItemData: Dictionary of [Text, Variant];
        LineNo: Integer;
    begin
        ExecutionTime := CurrentDateTime;

        if CustomerNo = '' then
            exit(false);

        if not CustomerExists(CustomerNo) then
            exit(false);

        try
            // Create sales header
            SalesHeader.Init();
            SalesHeader."Document Type" := SalesHeader."Document Type"::Order;
            SalesHeader."No." := '';
            SalesHeader.Insert(true);
            CreatedDocumentNo := SalesHeader."No.";

            SalesHeader.Validate("Sell-to Customer No.", CustomerNo);
            if OrderDate <> 0D then
                SalesHeader.Validate("Order Date", OrderDate);
            SalesHeader.Modify(true);

            // Add items
            LineNo := 10000;
            foreach ItemData in Items do begin
                if not CreateSalesLine(SalesHeader, ItemData, LineNo) then
                    exit(false);
                LineNo += 10000;
            end;

            // Update command data with result
            CommandData.Set('CreatedDocumentNo', CreatedDocumentNo);

            exit(true);

        except
            exit(false);
        end;
    end;

    procedure GetCommandName(): Text[50]
    begin
        exit('CREATE_SALES_DOCUMENT');
    end;

    procedure GetDescription(): Text[250]
    begin
        exit(StrSubstNo('Create sales document for customer %1 with %2 items', CustomerNo, Items.Count));
    end;

    procedure CanUndo(): Boolean
    begin
        exit(CreatedDocumentNo <> '');
    end;

    procedure Undo(): Boolean
    var
        SalesHeader: Record "Sales Header";
    begin
        if CreatedDocumentNo = '' then
            exit(false);

        if SalesHeader.Get(SalesHeader."Document Type"::Order, CreatedDocumentNo) then begin
            // Only allow undo if not posted
            if SalesHeader.Status = SalesHeader.Status::Open then begin
                SalesHeader.Delete(true);
                exit(true);
            end;
        end;

        exit(false);
    end;

    procedure GetExecutionTime(): DateTime
    begin
        exit(ExecutionTime);
    end;

    procedure GetCommandData(): Dictionary of [Text, Variant]
    begin
        exit(CommandData);
    end;

    local procedure CustomerExists(CustomerNo: Code[20]): Boolean
    var
        Customer: Record Customer;
    begin
        exit(Customer.Get(CustomerNo));
    end;

    local procedure CreateSalesLine(SalesHeader: Record "Sales Header"; ItemData: Dictionary of [Text, Variant]; LineNo: Integer): Boolean
    var
        SalesLine: Record "Sales Line";
        ItemNo: Code[20];
        Quantity: Decimal;
        UnitPrice: Decimal;
    begin
        try
            SalesLine.Init();
            SalesLine."Document Type" := SalesHeader."Document Type";
            SalesLine."Document No." := SalesHeader."No.";
            SalesLine."Line No." := LineNo;
            SalesLine.Insert(true);

            if ItemData.Get('ItemNo', ItemNo) then
                SalesLine.Validate("No.", ItemNo);

            if ItemData.Get('Quantity', Quantity) then
                SalesLine.Validate(Quantity, Quantity);

            if ItemData.Get('UnitPrice', UnitPrice) then
                SalesLine.Validate("Unit Price", UnitPrice);

            SalesLine.Modify(true);
            exit(true);

        except
            exit(false);
        end;
    end;
}
```

## Batch Processing Command

```al
// Command for batch operations
codeunit 50702 "Batch Update Prices Cmd" implements ICommand
{
    var
        ItemCategoryCode: Code[20];
        PriceAdjustmentPercent: Decimal;
        EffectiveDate: Date;
        ProcessedItems: List of [Code[20]];
        OriginalPrices: Dictionary of [Code[20], Decimal];
        ExecutionTime: DateTime;

    procedure Initialize(CategoryCode: Code[20]; AdjustmentPercent: Decimal; NewEffectiveDate: Date)
    begin
        ItemCategoryCode := CategoryCode;
        PriceAdjustmentPercent := AdjustmentPercent;
        EffectiveDate := NewEffectiveDate;
        Clear(ProcessedItems);
        Clear(OriginalPrices);
    end;

    procedure Execute(): Boolean
    var
        Item: Record Item;
        NewPrice: Decimal;
        ProcessedCount: Integer;
    begin
        ExecutionTime := CurrentDateTime;

        if ItemCategoryCode = '' then
            exit(false);

        if PriceAdjustmentPercent = 0 then
            exit(false);

        try
            Item.SetRange("Item Category Code", ItemCategoryCode);
            Item.SetRange(Blocked, false);

            if Item.FindSet() then
                repeat
                    // Store original price for undo capability
                    OriginalPrices.Set(Item."No.", Item."Unit Price");

                    // Calculate new price
                    NewPrice := Item."Unit Price" * (1 + PriceAdjustmentPercent / 100);
                    NewPrice := Round(NewPrice, 0.01);

                    // Update price
                    Item.Validate("Unit Price", NewPrice);
                    Item.Modify(true);

                    ProcessedItems.Add(Item."No.");
                    ProcessedCount += 1;

                    // Commit periodically for large batches
                    if ProcessedCount mod 100 = 0 then
                        Commit();

                until Item.Next() = 0;

            exit(true);

        except
            exit(false);
        end;
    end;

    procedure GetCommandName(): Text[50]
    begin
        exit('BATCH_UPDATE_PRICES');
    end;

    procedure GetDescription(): Text[250]
    begin
        exit(StrSubstNo('Update prices for category %1 by %2%', ItemCategoryCode, PriceAdjustmentPercent));
    end;

    procedure CanUndo(): Boolean
    begin
        exit(ProcessedItems.Count > 0);
    end;

    procedure Undo(): Boolean
    var
        Item: Record Item;
        ItemNo: Code[20];
        OriginalPrice: Decimal;
        UndoCount: Integer;
    begin
        try
            foreach ItemNo in ProcessedItems do begin
                if Item.Get(ItemNo) and OriginalPrices.Get(ItemNo, OriginalPrice) then begin
                    Item.Validate("Unit Price", OriginalPrice);
                    Item.Modify(true);
                    UndoCount += 1;

                    if UndoCount mod 100 = 0 then
                        Commit();
                end;
            end;

            exit(true);

        except
            exit(false);
        end;
    end;

    procedure GetExecutionTime(): DateTime
    begin
        exit(ExecutionTime);
    end;

    procedure GetCommandData(): Dictionary of [Text, Variant]
    var
        CommandData: Dictionary of [Text, Variant];
    begin
        CommandData.Set('ItemCategory', ItemCategoryCode);
        CommandData.Set('AdjustmentPercent', PriceAdjustmentPercent);
        CommandData.Set('EffectiveDate', EffectiveDate);
        CommandData.Set('ProcessedCount', ProcessedItems.Count);

        exit(CommandData);
    end;
}
```

## Priority Queue Implementation

```al
// Priority queue for command processing
codeunit 50703 "Priority Command Queue"
{
    var
        HighPriorityQueue: List of [Interface ICommand];
        NormalPriorityQueue: List of [Interface ICommand];
        LowPriorityQueue: List of [Interface ICommand];

    procedure EnqueueCommand(Command: Interface ICommand; Priority: Option High,Normal,Low)
    begin
        case Priority of
            Priority::High:
                HighPriorityQueue.Add(Command);
            Priority::Normal:
                NormalPriorityQueue.Add(Command);
            Priority::Low:
                LowPriorityQueue.Add(Command);
        end;

        LogCommandEnqueued(Command, Priority);
    end;

    procedure DequeueNextCommand(): Interface ICommand
    var
        EmptyCommand: Interface ICommand;
    begin
        // Process high priority first
        if HighPriorityQueue.Count > 0 then begin
            EmptyCommand := HighPriorityQueue.Get(1);
            HighPriorityQueue.RemoveAt(1);
            exit(EmptyCommand);
        end;

        // Then normal priority
        if NormalPriorityQueue.Count > 0 then begin
            EmptyCommand := NormalPriorityQueue.Get(1);
            NormalPriorityQueue.RemoveAt(1);
            exit(EmptyCommand);
        end;

        // Finally low priority
        if LowPriorityQueue.Count > 0 then begin
            EmptyCommand := LowPriorityQueue.Get(1);
            LowPriorityQueue.RemoveAt(1);
            exit(EmptyCommand);
        end;

        exit(EmptyCommand);
    end;

    procedure HasCommands(): Boolean
    begin
        exit((HighPriorityQueue.Count > 0) or (NormalPriorityQueue.Count > 0) or (LowPriorityQueue.Count > 0));
    end;

    procedure GetQueueCounts(): Dictionary of [Text, Integer]
    var
        Counts: Dictionary of [Text, Integer];
    begin
        Counts.Set('High', HighPriorityQueue.Count);
        Counts.Set('Normal', NormalPriorityQueue.Count);
        Counts.Set('Low', LowPriorityQueue.Count);

        exit(Counts);
    end;

    procedure ProcessAllCommandsByPriority(): Integer
    var
        Command: Interface ICommand;
        ProcessedCount: Integer;
    begin
        ProcessedCount := 0;

        while HasCommands() do begin
            Command := DequeueNextCommand();
            if ExecuteCommand(Command) then
                ProcessedCount += 1;
        end;

        exit(ProcessedCount);
    end;

    local procedure ExecuteCommand(Command: Interface ICommand): Boolean
    begin
        try
            exit(Command.Execute());
        except
            LogCommandError(Command, GetLastErrorText);
            exit(false);
        end;
    end;

    local procedure LogCommandEnqueued(Command: Interface ICommand; Priority: Option)
    begin
        // Log command enqueue with priority
    end;

    local procedure LogCommandError(Command: Interface ICommand; ErrorText: Text)
    begin
        // Log command execution error
    end;
}
```

## Usage Example

```al
// Example of using the command queue system
codeunit 50704 "Command Queue Usage Example"
{
    procedure DemonstrateCommandQueue()
    var
        CommandQueue: Codeunit "Command Queue Manager";
        SalesDocCommand: Codeunit "Create Sales Document Cmd";
        PriceUpdateCommand: Codeunit "Batch Update Prices Cmd";
        Items: List of [Dictionary of [Text, Variant]];
        ItemData: Dictionary of [Text, Variant];
        QueueStatus: Dictionary of [Text, Variant];
        ProcessedCount: Integer;
    begin
        // Create sales document command
        ItemData.Set('ItemNo', 'ITEM001');
        ItemData.Set('Quantity', 5);
        ItemData.Set('UnitPrice', 25.00);
        Items.Add(ItemData);

        Clear(ItemData);
        ItemData.Set('ItemNo', 'ITEM002');
        ItemData.Set('Quantity', 10);
        Items.Add(ItemData);

        SalesDocCommand.Initialize('CUST001', Items, Today);

        // Create price update command
        PriceUpdateCommand.Initialize('ELECTRONICS', 5.0, Today);

        // Enqueue commands
        CommandQueue.EnqueueCommand(SalesDocCommand);
        CommandQueue.EnqueueCommand(PriceUpdateCommand);

        // Process all commands
        ProcessedCount := CommandQueue.ProcessAllCommands();

        // Check status
        QueueStatus := CommandQueue.GetQueueStatus();

        Message('Processed %1 commands. Queue status: %2', ProcessedCount, Format(QueueStatus));

        // Demonstrate undo
        if CommandQueue.UndoLastCommand() then
            Message('Last command undone successfully');
    end;
}
```

This comprehensive implementation provides a flexible command queue system with support for priority processing, undo operations, batch processing, and comprehensive logging for reliable asynchronous processing in Business Central.