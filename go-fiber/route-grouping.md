# Route Grouping

Group logically related routes together using the `app.Group` method instead of registering endpoints flatly on the root app.

```go
// Bad:
user.Post("/users/list", c.GetGlobalUserList)
user.Post("/users/options", c.GetGlobalUserOptions)
user.Get("/users/:id", c.GetGlobalUserDetail)
user.Post("/users", c.CreateUser)

// Good:
user := app.Group("/users")

user.Post("/list", c.GetGlobalUserList)
user.Post("/options", c.GetGlobalUserOptions)
user.Get("/:id", c.GetGlobalUserDetail)
user.Post("/", c.CreateUser)
```

This reduces path duplication, clarifies the bounded context, and simplifies applying middleware (such as authentication) to an entire group of routes.
