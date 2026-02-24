# Graceful Shutdown

Listen for OS interrupt signals to trigger a graceful `Shutdown()` procedure on your Fiber application. This prevents abruptly killing inflight HTTP requests when deploying updates.

```go
// Bad:
func main() {
    app := fiber.New()
    // ... setup routes

    // This runs forever until killed, dropping any active requests
    app.Listen(":3000")
}

// Good:
func main() {
    sigs := make(chan os.Signal, 1)
    signal.Notify(sigs, os.Interrupt, syscall.SIGTERM)

    app := fiber.New()
    // ... setup routes

    // 1. Run server in a goroutine
    go func() {
        if err := app.Listen(":3000"); err != nil {
            slog.Error("listen error", "err", err)
        }
    }()

    // 2. Block until signal received
    <-sigs 
    slog.Info("Shutting down gracefully...")

    // 3. Command app to finish ongoing requests and close listeners
    if err := app.Shutdown(); err != nil {
        slog.Error("shutdown error", "err", err)
    }
}
```
