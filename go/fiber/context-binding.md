# Context Binding

Use `c.BodyParser`, `c.QueryParser`, or `c.ParamsParser` combined with struct validation to decode request input instead of parsing raw JSON or string values manually.

```go
// Bad:
func CreateUser(c *fiber.Ctx) error {
    var raw map[string]interface{}
    if err := json.Unmarshal(c.Body(), &raw); err != nil {
        return err
    }
    name, ok := raw["name"].(string)
    // ...
}

// Good:
type CreateUserRequest struct {
    Name string `json:"name" validate:"required"`
}

func CreateUser(c *fiber.Ctx) error {
    var req CreateUserRequest
    if err := c.BodyParser(&req); err != nil {
        e := util_entity.ToHttpError(err)
        return util_helper.SendResponse(c, fiber.StatusBadRequest, nil, e.Message, nil)
    }
    
    // Perform thorough validation (e.g. using a library like go-playground/validator)
    // if err := validate.Struct(&req); err != nil { ... }

    // ...
}
```

This ensures strong typing for handler inputs and enables robust declarative validations using struct tags.
