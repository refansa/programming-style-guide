# Middleware Order

Mount fundamental cross-cutting middlewares such as `Logger` and `CORS` globally before application-specific route groupings or feature-level middlewares.

```go
// Good:
app := fiber.New()

// 1. Logging and overall observability
app.Use(logger.New())

// 2. Global safety configurations (e.g., CORS, security headers)
app.Use(cors.New())

// 3. Auth middleware
app.Use(middleware.AuthMiddleware())
```
