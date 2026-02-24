# HTTP Workflow Execution

When triggering Temporal workflows directly from Fiber HTTP handlers, avoid manually polling for results, dealing with context cancelations, and serializing native Temporal errors to JSON. 

Instead, leverage the `temporal_helper.ExecuteWorkflowWithDefaults()` utility. This naturally binds the workflow lifecycle to the HTTP lifecycle, returning perfectly formatted JSON responses.

```go
// Bad:
func CreateUser(c *fiber.Ctx) error {
	ts := c.ServiceManager.Get("TemporalService").(*service.TemporalService)

    // Manually managing execution and casting results
    we, err := ts.Client.ExecuteWorkflow(c.Context(), options, "CreateUserWorkflow", param)
    if err != nil {
        return c.Status(500).SendString(err.Error()) // Ugly raw string response
    }
    
    var result dto.GetGlobalUserResult
    if err := we.Get(c.Context(), &result); err != nil {
        return c.Status(500).SendString(err.Error()) 
    }

    return c.JSON(result)
}

// Good:
func CreateUser(c *fiber.Ctx) error {
	ts := c.ServiceManager.Get("TemporalService").(*service.TemporalService)
    var result dto.GetGlobalUserResult

	config := temporal_helper.ExecuteWorkflowConfig{
		WorkflowName:    constant.WorkflowCreateUser,
		SuccessMessage:  "Successfully created user",
		SuccessCode:     fiber.StatusOK,
		TemporalService: ts,
		Param:           param,
		ErrorHasData:    true,
	}

    // Handles execution, blocking (Get), and HTTP response rendering natively
	return temporal_helper.ExecuteWorkflowWithDefaults(c, config, &result)
}
```
