# Internal App Layout (`/internal/app`)

The `internal/app` directory acts as the **Bootstrap and Dependency Injection Layer**. 

You will typically find `http_app.go` here, which is responsible for initializing the framework environment before the application legally begins listening on a port.

## Responsibilities
1. **Dependency Injection:** Instantiating the global `service.MakeManager()` and passing references to configuration files, databases, and message brokers.
2. **Framework Setup:** Creating the core `fiber.New()` instance.
3. **Registration:** Mounting the master routes, Temporal workers, and global middlewares into the server context.
4. **Lifecycle Exposal:** Providing handles back to `main.go` so that graceful shutdown sequences (`app.Shutdown()`) can be triggered upon OS signals.

Keep this directory incredibly thin. Do not place route handlers or utility functions here.
