# Task Scheduler Pattern Implementation - AL Code Examples

This sample demonstrates implementing robust task scheduling patterns for workflow automation in Business Central.

## Task Definition Interface

```al
// Core task interface for scheduler operations
interface IScheduledTask
{
    procedure Execute(): Boolean;
    procedure GetTaskName(): Text[50];
    procedure GetDescription(): Text[250];
    procedure CanRetry(): Boolean;
    procedure GetMaxRetries(): Integer;
}
```

## Task Scheduler Engine

```al
// Main task scheduling engine
codeunit 50300 "Task Scheduler Engine"
{
    var
        TaskQueue: List of [Codeunit "Scheduled Task Entry"];
        ExecutingTasks: Dictionary of [Guid, Boolean];

    procedure ScheduleTask(Task: Interface IScheduledTask; ExecutionTime: DateTime; Priority: Integer)
    var
        TaskEntry: Codeunit "Scheduled Task Entry";
        TaskId: Guid;
    begin
        TaskId := CreateGuid();
        TaskEntry.Initialize(TaskId, Task, ExecutionTime, Priority);

        AddToQueue(TaskEntry);
        LogTaskScheduled(TaskId, Task.GetTaskName(), ExecutionTime);
    end;

    procedure ScheduleRecurringTask(Task: Interface IScheduledTask; FirstExecution: DateTime; RecurrencePattern: Text[50])
    var
        NextExecution: DateTime;
        TaskId: Guid;
    begin
        TaskId := CreateGuid();
        NextExecution := FirstExecution;

        repeat
            ScheduleTask(Task, NextExecution, 5); // Default priority
            NextExecution := CalculateNextExecution(NextExecution, RecurrencePattern);
        until NextExecution > CalcDate('1Y', Today); // Limit to 1 year ahead
    end;

    procedure ProcessScheduledTasks()
    var
        TaskEntry: Codeunit "Scheduled Task Entry";
        CurrentTime: DateTime;
    begin
        CurrentTime := CurrentDateTime;

        foreach TaskEntry in TaskQueue do begin
            if TaskEntry.IsReadyForExecution(CurrentTime) and not IsTaskExecuting(TaskEntry.GetTaskId()) then
                ExecuteTaskAsync(TaskEntry);
        end;
    end;

    local procedure ExecuteTaskAsync(TaskEntry: Codeunit "Scheduled Task Entry")
    var
        TaskId: Guid;
        ExecutionResult: Boolean;
    begin
        TaskId := TaskEntry.GetTaskId();
        ExecutingTasks.Set(TaskId, true);

        try
            ExecutionResult := TaskEntry.Execute();
            HandleTaskCompletion(TaskEntry, ExecutionResult);
        finally
            ExecutingTasks.Remove(TaskId);
        end;
    end;

    local procedure HandleTaskCompletion(TaskEntry: Codeunit "Scheduled Task Entry"; Success: Boolean)
    begin
        if Success then begin
            RemoveFromQueue(TaskEntry);
            LogTaskCompleted(TaskEntry.GetTaskId(), TaskEntry.GetTaskName());
        end else begin
            HandleTaskFailure(TaskEntry);
        end;
    end;

    local procedure HandleTaskFailure(TaskEntry: Codeunit "Scheduled Task Entry")
    var
        RetryCount: Integer;
        MaxRetries: Integer;
    begin
        RetryCount := TaskEntry.GetRetryCount();
        MaxRetries := TaskEntry.GetMaxRetries();

        if (RetryCount < MaxRetries) and TaskEntry.CanRetry() then begin
            TaskEntry.IncrementRetryCount();
            TaskEntry.SetNextExecutionTime(CalcDate('5M', CurrentDateTime)); // Retry in 5 minutes
            LogTaskRetry(TaskEntry.GetTaskId(), TaskEntry.GetTaskName(), RetryCount + 1);
        end else begin
            RemoveFromQueue(TaskEntry);
            LogTaskFailed(TaskEntry.GetTaskId(), TaskEntry.GetTaskName());
            HandleDeadLetter(TaskEntry);
        end;
    end;

    local procedure AddToQueue(TaskEntry: Codeunit "Scheduled Task Entry")
    begin
        TaskQueue.Add(TaskEntry);
        SortQueueByPriorityAndTime();
    end;

    local procedure RemoveFromQueue(TaskEntry: Codeunit "Scheduled Task Entry")
    var
        Index: Integer;
        CurrentEntry: Codeunit "Scheduled Task Entry";
    begin
        for Index := 1 to TaskQueue.Count do begin
            CurrentEntry := TaskQueue.Get(Index);
            if CurrentEntry.GetTaskId() = TaskEntry.GetTaskId() then begin
                TaskQueue.RemoveAt(Index);
                break;
            end;
        end;
    end;

    local procedure SortQueueByPriorityAndTime()
    begin
        // Simplified sorting - in production, implement proper priority queue
    end;

    local procedure IsTaskExecuting(TaskId: Guid): Boolean
    var
        IsExecuting: Boolean;
    begin
        exit(ExecutingTasks.Get(TaskId, IsExecuting) and IsExecuting);
    end;

    local procedure CalculateNextExecution(LastExecution: DateTime; Pattern: Text[50]): DateTime
    begin
        case Pattern of
            'DAILY':
                exit(LastExecution + 24 * 60 * 60 * 1000); // Add 24 hours
            'WEEKLY':
                exit(CalcDate('1W', DT2Date(LastExecution)));
            'MONTHLY':
                exit(CalcDate('1M', DT2Date(LastExecution)));
            else
                exit(LastExecution + 60 * 60 * 1000); // Default: hourly
        end;
    end;

    local procedure HandleDeadLetter(TaskEntry: Codeunit "Scheduled Task Entry")
    begin
        // Move failed tasks to dead letter queue for manual review
    end;

    local procedure LogTaskScheduled(TaskId: Guid; TaskName: Text[50]; ExecutionTime: DateTime)
    begin
        // Log scheduling event
    end;

    local procedure LogTaskCompleted(TaskId: Guid; TaskName: Text[50])
    begin
        // Log completion event
    end;

    local procedure LogTaskRetry(TaskId: Guid; TaskName: Text[50]; RetryAttempt: Integer)
    begin
        // Log retry event
    end;

    local procedure LogTaskFailed(TaskId: Guid; TaskName: Text[50])
    begin
        // Log failure event
    end;
}
```

## Scheduled Task Entry

```al
// Individual task entry management
codeunit 50301 "Scheduled Task Entry"
{
    var
        TaskId: Guid;
        Task: Interface IScheduledTask;
        ScheduledTime: DateTime;
        Priority: Integer;
        RetryCount: Integer;
        MaxRetries: Integer;
        NextExecutionTime: DateTime;

    procedure Initialize(NewTaskId: Guid; NewTask: Interface IScheduledTask; ExecutionTime: DateTime; TaskPriority: Integer)
    begin
        TaskId := NewTaskId;
        Task := NewTask;
        ScheduledTime := ExecutionTime;
        NextExecutionTime := ExecutionTime;
        Priority := TaskPriority;
        RetryCount := 0;
        MaxRetries := Task.GetMaxRetries();
    end;

    procedure Execute(): Boolean
    begin
        exit(Task.Execute());
    end;

    procedure IsReadyForExecution(CurrentTime: DateTime): Boolean
    begin
        exit(CurrentTime >= NextExecutionTime);
    end;

    procedure CanRetry(): Boolean
    begin
        exit(Task.CanRetry() and (RetryCount < MaxRetries));
    end;

    procedure IncrementRetryCount()
    begin
        RetryCount += 1;
    end;

    procedure SetNextExecutionTime(ExecutionTime: DateTime)
    begin
        NextExecutionTime := ExecutionTime;
    end;

    procedure GetTaskId(): Guid
    begin
        exit(TaskId);
    end;

    procedure GetTaskName(): Text[50]
    begin
        exit(Task.GetTaskName());
    end;

    procedure GetRetryCount(): Integer
    begin
        exit(RetryCount);
    end;

    procedure GetMaxRetries(): Integer
    begin
        exit(MaxRetries);
    end;

    procedure GetPriority(): Integer
    begin
        exit(Priority);
    end;

    procedure GetNextExecutionTime(): DateTime
    begin
        exit(NextExecutionTime);
    end;
}
```

## Sales Report Generation Task

```al
// Example task: Generate sales reports
codeunit 50302 "Sales Report Generation Task" implements IScheduledTask
{
    var
        ReportParameters: Dictionary of [Text, Variant];

    procedure Execute(): Boolean
    var
        SalesReportGeneration: Codeunit "Sales Report Generator";
        ReportId: Integer;
        FromDate: Date;
        ToDate: Date;
    begin
        // Extract parameters
        if not ReportParameters.Get('FromDate', FromDate) then
            FromDate := CalcDate('-1M', Today);

        if not ReportParameters.Get('ToDate', ToDate) then
            ToDate := Today;

        if not ReportParameters.Get('ReportId', ReportId) then
            ReportId := Report::"Sales - Invoice";

        // Generate report
        try
            SalesReportGeneration.GenerateReport(ReportId, FromDate, ToDate);
            NotifyReportCompletion(ReportId, FromDate, ToDate);
            exit(true);
        except
            LogReportError(GetLastErrorText);
            exit(false);
        end;
    end;

    procedure GetTaskName(): Text[50]
    begin
        exit('SALES_REPORT_GENERATION');
    end;

    procedure GetDescription(): Text[250]
    begin
        exit('Generates scheduled sales reports for management review');
    end;

    procedure CanRetry(): Boolean
    begin
        exit(true);
    end;

    procedure GetMaxRetries(): Integer
    begin
        exit(3);
    end;

    procedure SetParameters(Parameters: Dictionary of [Text, Variant])
    begin
        ReportParameters := Parameters;
    end;

    local procedure NotifyReportCompletion(ReportId: Integer; FromDate: Date; ToDate: Date)
    var
        EmailMessage: Codeunit "Email Message";
        Email: Codeunit Email;
    begin
        // Send email notification about report completion
        EmailMessage.Create(
            'manager@company.com',
            'Sales Report Generated',
            StrSubstNo('Sales report %1 for period %2 to %3 has been generated successfully.', ReportId, FromDate, ToDate)
        );

        Email.Send(EmailMessage);
    end;

    local procedure LogReportError(ErrorText: Text)
    begin
        // Log report generation error
    end;
}
```

## Inventory Reorder Task

```al
// Example task: Automatic inventory reordering
codeunit 50303 "Inventory Reorder Task" implements IScheduledTask
{
    procedure Execute(): Boolean
    var
        Item: Record Item;
        PurchaseHeader: Record "Purchase Header";
        PurchaseLine: Record "Purchase Line";
        ItemsProcessed: Integer;
    begin
        ItemsProcessed := 0;

        Item.SetRange(Blocked, false);
        Item.SetRange("Replenishment System", Item."Replenishment System"::Purchase);
        Item.SetFilter("Reorder Point", '>0');

        if Item.FindSet() then
            repeat
                Item.CalcFields(Inventory);
                if Item.Inventory <= Item."Reorder Point" then begin
                    if CreateReorderPurchaseOrder(Item) then
                        ItemsProcessed += 1;
                end;
            until Item.Next() = 0;

        // Log results
        Message('Automatic reordering completed. %1 items processed.', ItemsProcessed);
        exit(true);
    end;

    procedure GetTaskName(): Text[50]
    begin
        exit('INVENTORY_REORDER');
    end;

    procedure GetDescription(): Text[250]
    begin
        exit('Automatically creates purchase orders for items below reorder point');
    end;

    procedure CanRetry(): Boolean
    begin
        exit(true);
    end;

    procedure GetMaxRetries(): Integer
    begin
        exit(2);
    end;

    local procedure CreateReorderPurchaseOrder(Item: Record Item): Boolean
    var
        PurchaseHeader: Record "Purchase Header";
        PurchaseLine: Record "Purchase Line";
        VendorNo: Code[20];
        OrderQuantity: Decimal;
    begin
        // Get preferred vendor
        VendorNo := GetPreferredVendor(Item."No.");
        if VendorNo = '' then
            exit(false);

        // Calculate order quantity
        OrderQuantity := Item."Maximum Inventory" - Item.Inventory;
        if OrderQuantity <= 0 then
            exit(false);

        // Create purchase header
        PurchaseHeader.Init();
        PurchaseHeader."Document Type" := PurchaseHeader."Document Type"::Order;
        PurchaseHeader."No." := '';
        PurchaseHeader.Insert(true);
        PurchaseHeader.Validate("Buy-from Vendor No.", VendorNo);
        PurchaseHeader.Modify(true);

        // Create purchase line
        PurchaseLine.Init();
        PurchaseLine."Document Type" := PurchaseHeader."Document Type";
        PurchaseLine."Document No." := PurchaseHeader."No.";
        PurchaseLine."Line No." := 10000;
        PurchaseLine.Insert(true);
        PurchaseLine.Validate(Type, PurchaseLine.Type::Item);
        PurchaseLine.Validate("No.", Item."No.");
        PurchaseLine.Validate(Quantity, OrderQuantity);
        PurchaseLine.Modify(true);

        exit(true);
    end;

    local procedure GetPreferredVendor(ItemNo: Code[20]): Code[20]
    var
        ItemVendor: Record "Item Vendor";
    begin
        ItemVendor.SetRange("Item No.", ItemNo);
        ItemVendor.SetRange("Vendor No.", '<>''''');
        if ItemVendor.FindFirst() then
            exit(ItemVendor."Vendor No.");

        exit('');
    end;
}
```

## Task Scheduler Management

```al
// Management interface for task scheduler
page 50300 "Task Scheduler Management"
{
    PageType = List;
    ApplicationArea = All;
    UsageCategory = Administration;
    Caption = 'Task Scheduler Management';

    layout
    {
        area(Content)
        {
            group(SchedulerStatus)
            {
                Caption = 'Scheduler Status';
                field(IsRunning; SchedulerRunning)
                {
                    ApplicationArea = All;
                    Caption = 'Scheduler Running';
                    Editable = false;
                }
                field(ActiveTasks; ActiveTaskCount)
                {
                    ApplicationArea = All;
                    Caption = 'Active Tasks';
                    Editable = false;
                }
            }
        }
    }

    actions
    {
        area(Processing)
        {
            action(StartScheduler)
            {
                Caption = 'Start Scheduler';
                ApplicationArea = All;

                trigger OnAction()
                begin
                    StartTaskScheduler();
                end;
            }

            action(StopScheduler)
            {
                Caption = 'Stop Scheduler';
                ApplicationArea = All;

                trigger OnAction()
                begin
                    StopTaskScheduler();
                end;
            }

            action(ScheduleSalesReport)
            {
                Caption = 'Schedule Sales Report';
                ApplicationArea = All;

                trigger OnAction()
                begin
                    ScheduleSalesReportTask();
                end;
            }

            action(ScheduleInventoryReorder)
            {
                Caption = 'Schedule Inventory Reorder';
                ApplicationArea = All;

                trigger OnAction()
                begin
                    ScheduleInventoryReorderTask();
                end;
            }
        }
    }

    var
        TaskScheduler: Codeunit "Task Scheduler Engine";
        SchedulerRunning: Boolean;
        ActiveTaskCount: Integer;

    trigger OnOpenPage()
    begin
        UpdateSchedulerStatus();
    end;

    local procedure StartTaskScheduler()
    begin
        // Start background processing
        TaskScheduler.ProcessScheduledTasks();
        SchedulerRunning := true;
        UpdateSchedulerStatus();
        Message('Task scheduler started successfully.');
    end;

    local procedure StopTaskScheduler()
    begin
        SchedulerRunning := false;
        UpdateSchedulerStatus();
        Message('Task scheduler stopped.');
    end;

    local procedure ScheduleSalesReportTask()
    var
        SalesReportTask: Codeunit "Sales Report Generation Task";
        Parameters: Dictionary of [Text, Variant];
        ExecutionTime: DateTime;
    begin
        // Set parameters
        Parameters.Set('FromDate', CalcDate('-1M', Today));
        Parameters.Set('ToDate', Today);
        Parameters.Set('ReportId', Report::"Sales - Invoice");

        SalesReportTask.SetParameters(Parameters);

        // Schedule for tomorrow at 6 AM
        ExecutionTime := CreateDateTime(CalcDate('1D', Today), 060000T);
        TaskScheduler.ScheduleTask(SalesReportTask, ExecutionTime, 1);

        Message('Sales report scheduled for %1', ExecutionTime);
    end;

    local procedure ScheduleInventoryReorderTask()
    var
        InventoryReorderTask: Codeunit "Inventory Reorder Task";
        ExecutionTime: DateTime;
    begin
        // Schedule for daily execution at 8 AM
        ExecutionTime := CreateDateTime(CalcDate('1D', Today), 080000T);
        TaskScheduler.ScheduleRecurringTask(InventoryReorderTask, ExecutionTime, 'DAILY');

        Message('Inventory reorder task scheduled for daily execution at 8:00 AM');
    end;

    local procedure UpdateSchedulerStatus()
    begin
        // Update status fields
        ActiveTaskCount := GetActiveTaskCount();
    end;

    local procedure GetActiveTaskCount(): Integer
    begin
        // Return count of active tasks
        exit(5); // Placeholder
    end;
}
```

This comprehensive implementation provides a robust task scheduling system with error handling, retry mechanisms, and support for both one-time and recurring tasks in Business Central.