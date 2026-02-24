# Safe Runner

Always wrap your primary workflow logic inside `temporal_helper.SafeRunner()`.

Go's default behavior is to immediately crash the entire application when a panic occurs in a goroutine. In Temporal, panics inside a workflow execution will cause the worker to crash, leading to continuous crashing loops as Temporal repeatedly re-assigns the crashing workflow task to other running workers.

```go
// Bad:
// A nil pointer panic here will crash the Temporal worker process continually 
// until the workflow execution hits a timeout.
func CreateUserWorkflow(ctx workflow.Context, param CreateUserWorkflowParam) (res any, err error) {
   // ... some logic that panics
   return nil, nil
}

// Good:
// Safety wrapper traps the panic, converts it cleanly to a workflow error, 
// and fails the workflow naturally without bringing down the host worker.
func CreateUserWorkflow(ctx workflow.Context, param CreateUserWorkflowParam) (res any, err error) {
	return temporal_helper.SafeRunner(func() (res any, err error) {
        // ... all core workflow logic
        return nil, nil
    })
}
```

This single helper prevents one rogue buggy workflow definition from causing a cascading denial of service across all your worker fleet nodes.
