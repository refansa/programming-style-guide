# Dependency Injection

Pass functional dependencies directly to constructors or initialization functions (such as route registrars or controllers). Do not rely on packages querying unstructured global state or init-time singletons for business logic bindings.

```go
// Bad:
// Relying on a globally accessible variable holding db connections
func CreateUser(c *fiber.Ctx) error {
    db := globals.GetDB()
    // ...
}

// Good:
// Inject dependencies via a constructed `Managerer` or structurally
func SetupRoute(app *fiber.Group, serviceManager service.Managerer) {
	c := &UserController{
		ServiceManager: serviceManager,
	}

	app.Post("/", c.CreateUser)
}

func (c *UserController) CreateUser(ctx *fiber.Ctx) error {
    dbService := c.ServiceManager.Get("DBProvider")
    // ...
}
```

Explicitly injecting your service locators allows dependencies to be mocked easily for tests and prevents initialization timing bugs.
