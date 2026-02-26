# Service Manager

The `service.Managerer` interface is the central dependency injection (DI) container used to register, retrieve, and unregister services throughout the application lifecycle.

## Creating a Manager

```go
sm := service.MakeManager()
```

## Registering Services

Call `Register()` to add a service. The service's `Init()` method fires immediately upon registration.

```go
sm.Register("DBProvider", &provider.DBProvider{})
sm.Register("RedisService", service.MakeRedisService())

ts, _ := service.MakeTemporalService("iam-task-queue")
sm.Register("TemporalService", ts)
```

> If a service with the same name already exists, it will be overridden.

## Retrieving Services

Use `Get()` with a type assertion to retrieve the concrete interface. This will **panic** if the service was never registered.

```go
dbProvider := sm.Get("DBProvider").(provider.DBProviderer)
redisService := sm.Get("RedisService").(service.RedisServicer)
temporalService := sm.Get("TemporalService").(service.TemporalServicer)
```

## Unregistering Services

Call `Unregister()` during graceful shutdown. The service's `Close()` method fires automatically. Safe to call multiple times even if not registered.

```go
sm.Unregister("TemporalService")
sm.Unregister("RedisService")
sm.Unregister("DBProvider")
```

## The Servicer Contract

Any service registered into the manager must implement the `Servicer` interface:

```go
type Servicer interface {
    Init()  // Called on Register(). Must panic on fatal errors.
    Close() // Called on Unregister(). Must NOT panic.
}
```
