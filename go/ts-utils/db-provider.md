# DB Provider

`DBProviderer` is a factory interface that creates database service instances on-the-fly. It is the standard way to obtain `PostgreSQLServicer`, `MongoDBServicer`, or `RedisServicer` instances inside Temporal Activities.

## Registering the Provider

Register once during application bootstrap in `main.go`:

```go
sm.Register("DBProvider", &provider.DBProvider{})
```

## Retrieving Inside Activities

Type-assert from the service manager:

```go
dbProvider := a.ServiceManager.Get("DBProvider").(provider.DBProviderer)
```

## Creating Database Services

### PostgreSQL (Per-Tenant)

The most common usage pattern creates a per-tenant PostgreSQL service using a company-specific database name:

```go
dbName := db.IAMDBName.CompanyDBName(clientId)
pgService := dbProvider.MakePostgreSQLService(dbName)
```

### Redis

```go
redisService := dbProvider.MakeRedisService()
```

### MongoDB

```go
mongoService := dbProvider.MakeMongoDBService(db.SomeDBName, "collection_name")
```

## Interface

```go
type DBProviderer interface {
    Init()
    MakeMongoDBService(dbName db.DBName, collection string) service.MongoDBServicer
    MakePostgreSQLService(dbName db.DBName, options ...db.PostgreSQLServiceOptions) service.PostgreSQLServicer
    MakeRedisService(options ...db.RedisServiceOptions) service.RedisServicer
    Close()
}
```
