# Workflow Compensation (Sagas)

When building workflows that orchestrate multiple side-effects (e.g. creating records across different databases, calling external APIs), you must use the Saga pattern to undo completed steps if a subsequent step fails. 

Use the `service.NewWorkflowCompensation()` utility to manage these undo operations cleanly.

1. **Defer Execution:** Always `defer` the `.Compensate()` method at the top using a `DisconnectedContext`. If the workflow fails (even via timeout or cancelation), the compensation block will reliably run.
2. **Add After Success:** Only call `.AddCompensation()` _after_ an activity executes successfully. Do not add compensations for activities that failed or haven't run yet.

```go
// Good:
func CreateUserWorkflow(ctx workflow.Context, param CreateUserWorkflowParam) (res any, err error) {
	userCompensation := service.NewWorkflowCompensation()
	userCtx := workflow.WithActivityOptions(ctx, temporal_helper.DefaultActivityOptions())

    // 1. Defer the rollback using a disconnected context so it cannot be cancelled
	defer func() {
		if err != nil {
			disconnectedUserCtx, _ := workflow.NewDisconnectedContext(userCtx)
			userCompensation.Compensate(disconnectedUserCtx, false)
		}
	}()

	return temporal_helper.SafeRunner(func() (res any, err error) {
        
        // 2. Execute forward action
        var result user_activity.CreateUserDataActivityResult
		err = workflow.ExecuteActivity(userCtx, constant.ActivityCreateUserData, param).Get(userCtx, &result)
        if err != nil {
            return nil, err // Fails here, compensation runs (but it is empty, so nothing happens)
        }

        // 3. Add compensation ONLY because it succeeded
        userCompensation.AddCompensation(constant.ActivityDeleteUserData, user_activity.DeleteUserDataActivityParam{
            UserID: result.UserID,
        })

        // 4. Do next step... if THIS fails, the deferred block will trigger ActivityDeleteUserData
        err = workflow.ExecuteActivity(userCtx, constant.ActivityCreateUserLog, user_activity.CreateUserLogActivityParam{
            UserID: result.UserID,
            Action: "Create",
        }).Get(userCtx, nil)
        if err != nil {
            return nil, err 
        }

		return nil, nil
	})
}
```
