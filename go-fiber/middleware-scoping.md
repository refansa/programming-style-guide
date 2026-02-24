# Middleware Scoping

Apply middlewares sequentially taking advantage of the execution flow inside route registration. Mount broad public routes first, followed by restricted application-level middlewares, and seamlessly mount the protected routes below them.

This prevents the need to scatter repetitive authorization middleware calls across hundreds of endpoint groupings.

```go
// Bad:
// Manually passing `AuthMiddleware` to every protected group
func RegisterRoutes(app *fiber.App) {
	auth_route.SetupRoute(app)
	user_route.SetupRoute(app.Group("/users", middleware.AuthMiddleware()))
	position_route.SetupRoute(app.Group("/positions", middleware.AuthMiddleware()))
}

// Good:
// Utilizing the route builder's sequential order
func RegisterRoutes(app *fiber.App) {
    // 1. Unprotected public routes
	auth_route.SetupRoute(app)

    // 2. Global application-level protection boundary
	app.Use(middleware.AuthMiddleware())

    // 3. Protected groupings (no repeating middleware required)
	user_route.SetupRoute(app)
	position_route.SetupRoute(app)
	division_route.SetupRoute(app)
}
```
