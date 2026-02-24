# Activity Errors

Never return raw native driver errors (like `sql.ErrNoRows` or `pgx.ErrNoRows`) directly from Temporal Activities. 

Because handlers leverage `temporal_helper.ExecuteWorkflowWithDefaults` to map workflow output back to the client, an unhandled database error will serialize to the client as an ugly `Internal Server Error` (HTTP 500).

```go
// Bad:
func (a *Activities) GetUserActivity(ctx context.Context, ID string) (*model.User, error) {
    var user model.User
    err := a.DB.SelectOne(&user, query, args)
    if err != nil {
        // Will cause HTTP 500 upstream if it's just a "Not Found" scenario
        return nil, err
    }
    return &user, nil
}

// Good:
func (a *Activities) GetUserActivity(ctx context.Context, ID string) (*model.User, error) {
    var user model.User
    err := iamDBService.SelectOne(&user, ctx, query, args...)
    
	if err != nil {
		if errors.Is(err, sql.ErrNoRows) || errors.Is(err, pgx.ErrNoRows) {
            // Evaluates to a clean HTTP 404 upstream
			return nil, entity.WorkflowNotFound("User not found", true)
		}
        
        // Wrap unexpected errors so they are clearly marked 
		return nil, entity.WrapHttpError(&entity.HttpError{
			Code:    fiber.StatusInternalServerError,
			Message: "Failed to fetch user by ID",
		}, true) 
	}

    return &user, nil
}
```

Always use `entity.WorkflowNotFound(..., true)` or `entity.WorkflowErrorWrapper(...)` inside activities (or inside workflow logic right after activity checks) to correctly bridge domain failures with the HTTP presentation layer.
