# Business Process Templates Sample

## Overview
This sample demonstrates business process template patterns including configurable workflows, state management, and business rule integration for standardized process execution.

## Basic Process Template Structure

```al
table 50600 "Business Process Template"
{
    fields
    {
        field(1; "Template Code"; Code[20]) { }
        field(2; "Template Name"; Text[100]) { }
        field(3; "Process Type"; Enum "Business Process Type") { }
        field(4; "Status"; Enum "Template Status") { }
        field(5; "Version"; Code[10]) { }
        field(6; "Configuration Data"; Blob) { }
        field(7; "Default Values"; Blob) { }
    }

    keys
    {
        key(PK; "Template Code") { Clustered = true; }
    }
}

table 50601 "Process Template Step"
{
    fields
    {
        field(1; "Template Code"; Code[20]) { }
        field(2; "Step No."; Integer) { }
        field(3; "Step Name"; Text[100]) { }
        field(4; "Step Type"; Enum "Process Step Type") { }
        field(5; "Execution Order"; Integer) { }
        field(6; "Required"; Boolean) { }
        field(7; "Condition"; Text[250]) { }
        field(8; "Configuration"; Blob) { }
    }

    keys
    {
        key(PK; "Template Code", "Step No.") { Clustered = true; }
        key(ExecutionOrder; "Template Code", "Execution Order") { }
    }
}
```

## Sales Order Process Template

```al
codeunit 50600 "Sales Order Process Template"
{
    procedure ExecuteTemplate(TemplateCode: Code[20]; var SalesHeader: Record "Sales Header"): Boolean
    var
        ProcessTemplate: Record "Business Process Template";
        TemplateStep: Record "Process Template Step";
        ProcessContext: Record "Process Execution Context";
    begin
        if not ProcessTemplate.Get(TemplateCode) then
            Error('Process template %1 not found', TemplateCode);

        ProcessContext := InitializeProcessContext(TemplateCode, SalesHeader);

        TemplateStep.SetRange("Template Code", TemplateCode);
        TemplateStep.SetCurrentKey("Template Code", "Execution Order");

        if TemplateStep.FindSet() then
            repeat
                if ShouldExecuteStep(TemplateStep, SalesHeader, ProcessContext) then begin
                    if not ExecuteProcessStep(TemplateStep, SalesHeader, ProcessContext) then begin
                        HandleStepFailure(TemplateStep, SalesHeader, ProcessContext);
                        if TemplateStep.Required then
                            exit(false);
                    end else
                        LogStepSuccess(TemplateStep, ProcessContext);
                end;
            until TemplateStep.Next() = 0;

        FinalizeProcess(ProcessContext, SalesHeader);
        exit(true);
    end;

    local procedure InitializeProcessContext(TemplateCode: Code[20]; SalesHeader: Record "Sales Header"): Record "Process Execution Context"
    var
        ProcessContext: Record "Process Execution Context";
    begin
        ProcessContext.Init();
        ProcessContext."Process ID" := CreateGuid();
        ProcessContext."Template Code" := TemplateCode;
        ProcessContext."Document Type" := Format(SalesHeader."Document Type");
        ProcessContext."Document No." := SalesHeader."No.";
        ProcessContext."Started DateTime" := CurrentDateTime;
        ProcessContext."User ID" := UserId;
        ProcessContext.Status := ProcessContext.Status::Running;
        ProcessContext.Insert(true);

        exit(ProcessContext);
    end;

    local procedure ExecuteProcessStep(TemplateStep: Record "Process Template Step"; var SalesHeader: Record "Sales Header"; var ProcessContext: Record "Process Execution Context"): Boolean
    var
        StepProcessor: Interface "Process Step Processor";
    begin
        StepProcessor := GetStepProcessor(TemplateStep."Step Type");
        exit(StepProcessor.ExecuteStep(TemplateStep, SalesHeader, ProcessContext));
    end;

    local procedure GetStepProcessor(StepType: Enum "Process Step Type"): Interface "Process Step Processor"
    var
        ValidationProcessor: Codeunit "Validation Step Processor";
        CalculationProcessor: Codeunit "Calculation Step Processor";
        ApprovalProcessor: Codeunit "Approval Step Processor";
        NotificationProcessor: Codeunit "Notification Step Processor";
    begin
        case StepType of
            StepType::Validation:
                exit(ValidationProcessor);
            StepType::Calculation:
                exit(CalculationProcessor);
            StepType::Approval:
                exit(ApprovalProcessor);
            StepType::Notification:
                exit(NotificationProcessor);
        end;
    end;
}
```

## Approval Process Template

```al
codeunit 50601 "Approval Process Template"
{
    procedure ProcessApprovalWorkflow(DocumentType: Enum "Approval Document Type"; DocumentNo: Code[20]; TemplateCode: Code[20]): Boolean
    var
        ApprovalEntry: Record "Approval Entry";
        ProcessContext: Record "Process Execution Context";
        WorkflowCompleted: Boolean;
    begin
        ProcessContext := InitializeApprovalProcess(DocumentType, DocumentNo, TemplateCode);

        CreateApprovalEntries(DocumentType, DocumentNo, TemplateCode);

        SendApprovalNotifications(DocumentType, DocumentNo);

        // Monitor approval progress
        WorkflowCompleted := MonitorApprovalProgress(ProcessContext);

        if WorkflowCompleted then
            FinalizeApprovalProcess(ProcessContext, DocumentType, DocumentNo)
        else
            HandleApprovalTimeout(ProcessContext, DocumentType, DocumentNo);

        exit(WorkflowCompleted);
    end;

    local procedure CreateApprovalEntries(DocumentType: Enum "Approval Document Type"; DocumentNo: Code[20]; TemplateCode: Code[20])
    var
        ApprovalTemplate: Record "Approval Process Template";
        ApprovalStep: Record "Approval Template Step";
        ApprovalEntry: Record "Approval Entry";
        UserSetup: Record "User Setup";
    begin
        ApprovalStep.SetRange("Template Code", TemplateCode);
        ApprovalStep.SetCurrentKey("Template Code", "Approval Level");

        if ApprovalStep.FindSet() then
            repeat
                if GetApproverForLevel(ApprovalStep."Approval Level", UserSetup) then begin
                    ApprovalEntry.Init();
                    ApprovalEntry."Table ID" := GetTableID(DocumentType);
                    ApprovalEntry."Document No." := DocumentNo;
                    ApprovalEntry."Sequence No." := GetNextSequenceNo();
                    ApprovalEntry."Approver ID" := UserSetup."User ID";
                    ApprovalEntry.Status := ApprovalEntry.Status::Created;
                    ApprovalEntry."Due Date" := CalcDate(ApprovalStep."Due Date Formula", Today);
                    ApprovalEntry.Insert(true);
                end;
            until ApprovalStep.Next() = 0;
    end;

    local procedure SendApprovalNotifications(DocumentType: Enum "Approval Document Type"; DocumentNo: Code[20])
    var
        ApprovalEntry: Record "Approval Entry";
        NotificationManagement: Codeunit "Notification Management";
    begin
        ApprovalEntry.SetRange("Document No.", DocumentNo);
        ApprovalEntry.SetRange(Status, ApprovalEntry.Status::Created);

        if ApprovalEntry.FindSet() then
            repeat
                NotificationManagement.SendApprovalRequest(ApprovalEntry);
                ApprovalEntry.Status := ApprovalEntry.Status::"Pending Approval";
                ApprovalEntry.Modify();
            until ApprovalEntry.Next() = 0;
    end;
}
```

## Invoice Processing Template

```al
codeunit 50602 "Invoice Processing Template"
{
    procedure ProcessInvoiceWorkflow(var SalesHeader: Record "Sales Header"; TemplateCode: Code[20]): Boolean
    var
        InvoiceSteps: Record "Invoice Process Step";
        ProcessResult: Boolean;
    begin
        ValidateInvoiceData(SalesHeader);

        InvoiceSteps.SetRange("Template Code", TemplateCode);
        InvoiceSteps.SetCurrentKey("Template Code", "Processing Order");

        if InvoiceSteps.FindSet() then
            repeat
                ProcessResult := ExecuteInvoiceStep(InvoiceSteps, SalesHeader);
                UpdateStepStatus(InvoiceSteps, ProcessResult);

                if not ProcessResult and InvoiceSteps."Critical Step" then
                    exit(false);

            until InvoiceSteps.Next() = 0;

        CompleteInvoiceProcessing(SalesHeader);
        exit(true);
    end;

    local procedure ExecuteInvoiceStep(InvoiceStep: Record "Invoice Process Step"; var SalesHeader: Record "Sales Header"): Boolean
    begin
        case InvoiceStep."Step Type" of
            InvoiceStep."Step Type"::"Credit Check":
                exit(PerformCreditCheck(SalesHeader));
            InvoiceStep."Step Type"::"Pricing Validation":
                exit(ValidatePricing(SalesHeader));
            InvoiceStep."Step Type"::"Tax Calculation":
                exit(CalculateTaxes(SalesHeader));
            InvoiceStep."Step Type"::"Discount Application":
                exit(ApplyDiscounts(SalesHeader));
            InvoiceStep."Step Type"::"Final Validation":
                exit(PerformFinalValidation(SalesHeader));
            InvoiceStep."Step Type"::"Document Generation":
                exit(GenerateInvoiceDocument(SalesHeader));
        end;

        exit(false);
    end;

    local procedure PerformCreditCheck(SalesHeader: Record "Sales Header"): Boolean
    var
        Customer: Record Customer;
        CreditCheck: Codeunit "Credit Management";
    begin
        Customer.Get(SalesHeader."Bill-to Customer No.");
        exit(CreditCheck.CheckCustomerCredit(Customer, SalesHeader."Amount Including VAT"));
    end;

    local procedure ValidatePricing(var SalesHeader: Record "Sales Header"): Boolean
    var
        SalesLine: Record "Sales Line";
        PriceCalculation: Codeunit "Price Calculation Mgt.";
    begin
        SalesLine.SetRange("Document Type", SalesHeader."Document Type");
        SalesLine.SetRange("Document No.", SalesHeader."No.");

        if SalesLine.FindSet() then
            repeat
                PriceCalculation.ApplyPrice(SalesLine);
                SalesLine.Modify();
            until SalesLine.Next() = 0;

        exit(true);
    end;
}
```

## Configurable Workflow Engine

```al
codeunit 50603 "Configurable Workflow Engine"
{
    procedure ExecuteConfigurableWorkflow(WorkflowCode: Code[20]; RecordVariant: Variant): Boolean
    var
        WorkflowTemplate: Record "Workflow Template";
        WorkflowInstance: Record "Workflow Instance";
        ExecutionSuccessful: Boolean;
    begin
        if not WorkflowTemplate.Get(WorkflowCode) then
            Error('Workflow template %1 not found', WorkflowCode);

        WorkflowInstance := CreateWorkflowInstance(WorkflowTemplate, RecordVariant);

        ExecutionSuccessful := ExecuteWorkflowSteps(WorkflowInstance);

        if ExecutionSuccessful then
            CompleteWorkflowInstance(WorkflowInstance)
        else
            HandleWorkflowFailure(WorkflowInstance);

        exit(ExecutionSuccessful);
    end;

    local procedure ExecuteWorkflowSteps(var WorkflowInstance: Record "Workflow Instance"): Boolean
    var
        WorkflowStep: Record "Workflow Template Step";
        StepExecutor: Codeunit "Workflow Step Executor";
        StepResult: Boolean;
        AllStepsSuccessful: Boolean;
    begin
        AllStepsSuccessful := true;

        WorkflowStep.SetRange("Template Code", WorkflowInstance."Template Code");
        WorkflowStep.SetCurrentKey("Template Code", "Execution Order");

        if WorkflowStep.FindSet() then
            repeat
                if EvaluateStepCondition(WorkflowStep, WorkflowInstance) then begin
                    UpdateWorkflowInstanceStatus(WorkflowInstance, 'Processing step: ' + WorkflowStep."Step Name");

                    StepResult := StepExecutor.ExecuteStep(WorkflowStep, WorkflowInstance);

                    LogStepExecution(WorkflowStep, WorkflowInstance, StepResult);

                    if not StepResult then begin
                        AllStepsSuccessful := false;
                        if WorkflowStep."Stop on Error" then
                            exit(false);
                    end;
                end;
            until WorkflowStep.Next() = 0;

        exit(AllStepsSuccessful);
    end;

    local procedure EvaluateStepCondition(WorkflowStep: Record "Workflow Template Step"; WorkflowInstance: Record "Workflow Instance"): Boolean
    var
        ConditionEvaluator: Codeunit "Workflow Condition Evaluator";
    begin
        if WorkflowStep.Condition = '' then
            exit(true);

        exit(ConditionEvaluator.EvaluateCondition(WorkflowStep.Condition, WorkflowInstance));
    end;
}
```

## State Management Template

```al
codeunit 50604 "Process State Manager"
{
    var
        ProcessStateStack: List of [Text];

    procedure ManageProcessState(var ProcessInstance: Record "Process Instance"; NewState: Enum "Process State"): Boolean
    var
        StateTransition: Record "Process State Transition";
        PreviousState: Enum "Process State";
    begin
        PreviousState := ProcessInstance.State;

        if not ValidateStateTransition(PreviousState, NewState) then
            exit(false);

        // Save current state to stack for potential rollback
        SaveStateToStack(ProcessInstance, PreviousState);

        // Update process state
        ProcessInstance.State := NewState;
        ProcessInstance."State Changed DateTime" := CurrentDateTime;
        ProcessInstance."State Changed By" := UserId;

        // Log state transition
        LogStateTransition(ProcessInstance, PreviousState, NewState);

        ProcessInstance.Modify(true);

        // Execute state-specific logic
        ExecuteStateActions(ProcessInstance, NewState);

        exit(true);
    end;

    procedure RollbackToLastState(var ProcessInstance: Record "Process Instance"): Boolean
    var
        LastState: Text;
        StateEnum: Enum "Process State";
    begin
        if ProcessStateStack.Count = 0 then
            exit(false);

        LastState := ProcessStateStack.Get(ProcessStateStack.Count);
        ProcessStateStack.RemoveAt(ProcessStateStack.Count);

        Evaluate(StateEnum, LastState);

        ProcessInstance.State := StateEnum;
        ProcessInstance."State Changed DateTime" := CurrentDateTime;
        ProcessInstance."State Changed By" := UserId;
        ProcessInstance.Modify(true);

        exit(true);
    end;

    local procedure ValidateStateTransition(FromState: Enum "Process State"; ToState: Enum "Process State"): Boolean
    var
        StateTransitionRule: Record "Process State Transition Rule";
    begin
        StateTransitionRule.SetRange("From State", FromState);
        StateTransitionRule.SetRange("To State", ToState);
        StateTransitionRule.SetRange(Allowed, true);

        exit(not StateTransitionRule.IsEmpty);
    end;

    local procedure ExecuteStateActions(ProcessInstance: Record "Process Instance"; NewState: Enum "Process State")
    var
        StateAction: Record "Process State Action";
        ActionProcessor: Codeunit "Process Action Processor";
    begin
        StateAction.SetRange("Process State", NewState);
        StateAction.SetRange(Active, true);

        if StateAction.FindSet() then
            repeat
                ActionProcessor.ExecuteAction(StateAction, ProcessInstance);
            until StateAction.Next() = 0;
    end;
}
```

This sample demonstrates comprehensive business process template patterns including configurable workflows, approval processes, invoice processing, workflow engines, and state management for standardized and flexible business process execution.