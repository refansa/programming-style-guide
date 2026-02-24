# Child Workflows (Fire and Forget)

When a parent workflow must trigger an asynchronous side-effect (like sending a verification email) that shouldn't block the parent workflow's completion, spawn a "fire-and-forget" child workflow.

To do this securely without blocking, use `temporal_helper.FireAndForgetChildWorkflowOptions()`. Wait for the child workflow to *start executing* on the server, but do **not** wait for it to *complete*.

```go
// Good:
func CreateUserWorkflow(ctx workflow.Context, param CreateUserWorkflowParam) (res any, err error) {
    // ... complete essential user creation logic ...

	// 1. Prepare fire-and-forget options. 
    //    Pass a deterministic workflow ID to prevent duplicate side-effects on parent retries.
	childWFOpt := temporal_helper.FireAndForgetChildWorkflowOptions(
		ctx,
		fmt.Sprintf("send-user-verification-%d", createdUserId),
	)
	childCtx := workflow.WithChildOptions(ctx, childWFOpt)

    // 2. Dispatch the child workflow
	childWorkflowFuture := workflow.ExecuteChildWorkflow(
		childCtx,
		constant.WorkflowSendEmail,
		param,
	)

	// 3. Block ONLY until the child workflow has started executing.
    //    This guarantees Temporal has durably recorded the child's launch before the parent exits.
	var childWE workflow.Execution
	if err := childWorkflowFuture.GetChildWorkflowExecution().Get(ctx, &childWE); err != nil {
		return nil, err
	}

    // 4. Return immediately. Do not call childWorkflowFuture.Get(ctx, nil) here!
	return nil, nil
}
```

This ensures background tasks are reliably delivered to the Temporal queue, decoupling their potential slowness or failures from the core synchronous business flow.
