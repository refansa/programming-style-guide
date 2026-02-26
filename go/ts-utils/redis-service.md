# Redis Service

`RedisServicer` provides a thin wrapper around the `go-redis/redis` client for common key-value operations.

## Creating an Instance

```go
redisService := service.MakeRedisService()
sm.Register("RedisService", redisService)
```

Or via the DB Provider:

```go
dbProvider := sm.Get("DBProvider").(provider.DBProviderer)
redisService := dbProvider.MakeRedisService()
```

## Available Commands

### Set
```go
err := redisService.Set(ctx, "session:abc123", tokenJSON, 24*time.Hour)
```

### Get
```go
val, err := redisService.Get(ctx, "session:abc123")
```

### Del
```go
err := redisService.Del(ctx, "session:abc123", "session:def456")
```

### HGetAll
Retrieve all fields and values of a hash.

```go
result, err := redisService.HGetAll(ctx, "user:1:settings")
// result → map[string]string{"theme": "dark", "lang": "en"}
```

### Expire
Set a timeout on an existing key.

```go
err := redisService.Expire(ctx, "session:abc123", 1*time.Hour)
```

## Accessing the Raw Client

For operations not covered by the interface, access the underlying `*redis.Client` directly.

```go
client := redisService.GetClient()
```
