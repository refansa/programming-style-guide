# Parameter Validation

When handling path parameters or query strings that dictate domain logic (like database IDs), parse and strictly validate them at the very beginning of the controller function. Reject invalid inputs immediately before executing deeper logic.

```go
// Bad:
// Assuming strings are always valid IDs or pushing conversion deep into workflows.
func GetGlobalUserDetail(ctx *fiber.Ctx) error {
    userId := ctx.Params("id")
    // ... immediately passing userId string to services
}

// Good:
// Immediately converting, validating, and rejecting bad inputs.
func GetGlobalUserDetail(ctx *fiber.Ctx) error {
    userId := ctx.Params("id")
    
    // 1. Strictly parse to correct business domain type immediately
    userIdInt, err := strconv.Atoi(userId)
    if err != nil {
        return util_entity.BadRequest("INVALID_USER_ID")
    }

    // ... continue with userIdInt safely
}
```

Eagerly discarding bad input avoids unnecessary resource allocation, database calls, or complex cross-layer debugging when incorrect string formats inevitably trigger downstream failures.
