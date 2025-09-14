# Manual Binding for Conditional Subscribers - AL Code Samples

## Basic Manual Binding Pattern

```al
codeunit 50400 "Manual Event Binding Manager"
{
    var
        SubscriberBindings: Dictionary of [Text, Boolean];

    procedure InitializeConditionalBindings()
    var
        CompanySetup: Record "Company Information";
        FeatureManagement: Record "Feature Key";
    begin
        CompanySetup.Get();
        
        if ShouldBindCustomerValidation(CompanySetup) then
            BindCustomerValidationSubscriber()
        else
            UnbindCustomerValidationSubscriber();
            
        if FeatureManagement.Get('ADVANCED_INVENTORY') and FeatureManagement.Enabled then
            BindInventoryTrackingSubscriber()
        else
            UnbindInventoryTrackingSubscriber();
    end;

    local procedure BindCustomerValidationSubscriber()
    var
        BindingKey: Text;
    begin
        BindingKey := 'CustomerValidation';
        
        if not IsSubscriberBound(BindingKey) then begin
            BindSubscription(Codeunit::"Customer Validation Subscriber");
            SubscriberBindings.Set(BindingKey, true);
        end;
    end;

    local procedure IsSubscriberBound(BindingKey: Text): Boolean
    var
        IsBound: Boolean;
    begin
        if SubscriberBindings.Get(BindingKey, IsBound) then
            exit(IsBound)
        else
            exit(false);
    end;
}
```

## Context-Aware Binding

```al
codeunit 50401 "Context-Aware Binding Manager"
{
    procedure InitializeUserContextBindings()
    var
        UserSetup: Record "User Setup";
    begin
        if UserSetup.Get(UserId) then begin
            if HasAuditPermissions(UserId) then
                BindAuditTrailSubscriber()
            else
                UnbindAuditTrailSubscriber();
                
            if HasApprovalPermissions(UserId) then
                BindApprovalSubscriber()
            else
                UnbindApprovalSubscriber();
        end;
    end;
    
    local procedure HasAuditPermissions(UserID: Text): Boolean
    var
        AccessControl: Record "Access Control";
    begin
        AccessControl.SetRange("User Name", UserID);
        AccessControl.SetRange("Role ID", 'AUDITOR');
        exit(not AccessControl.IsEmpty);
    end;
}
```