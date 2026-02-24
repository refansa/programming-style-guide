# Response Format

Maintain a consistent response body structure across all endpoints by using a centralized response helper function. A heterogeneous response format makes it very difficult for clients to parse your server's JSON reliably.

Do not build the JSON map manually or use inline anonymous structs within the handler.

```go
// Bad:
func GetUser(c *fiber.Ctx) error {
    ...
    return c.Status(fiber.StatusOK).JSON(fiber.Map{
        "status":  "success",
        "message": "Successfully retrieved user list",
        "data":    userList,
    })
}

// Good:
// Rely on a shared utility library function like `helper.SendResponse` to guarantee a 
// uniform output payload (e.g. data, message, errors) across the entire application.
func GetUser(c *fiber.Ctx) error {
    ...

    return helper.SendResponse(c, fiber.StatusOK, user, "Successfully retrieved user list", nil)
}

// Good (Temporal Workflow):
// When executing Temporal workflows, use `temporal_helper.ExecuteWorkflowWithDefaults` 
// to automatically handle execution, waiting, mapping results, and generating standard responses.
func GetGlobalUserList(c *fiber.Ctx) error {
    ...
    
    config := temporal_helper.ExecuteWorkflowConfig{
		WorkflowName:    constant.WorkflowGetGlobalUserList,
		SuccessMessage:  "Successfully get global user list",
		SuccessCode:     fiber.StatusOK,
		TemporalService: ts,
		Param:           param,
		ErrorHasData:    true,
	}

	return temporal_helper.ExecuteWorkflowWithDefaults(c, config, &result)
}
```

This pattern guarantees that every API response correctly maps standard fields like data, HTTP code, success messages, and validation payloads securely without duplication.
