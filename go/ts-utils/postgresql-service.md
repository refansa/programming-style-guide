# PostgreSQL Service

`PostgreSQLServicer` provides high-level shorthand methods for executing SQL queries against PostgreSQL. It wraps `pgx/v5` and integrates with the `sql_query` builder for filter-based operations.

## Obtaining an Instance

PostgreSQL services are created through the `DBProviderer` factory, typically inside Temporal Activities.

```go
dbProvider := a.ServiceManager.Get("DBProvider").(provider.DBProviderer)
dbName := db.IAMDBName.CompanyDBName(clientId)
postgreSqlService := dbProvider.MakePostgreSQLService(dbName)
```

## Select

### Raw Query
```go
var user model.User
query, args, _ := sql_query.NewSQLSelectBuilder[model.User](db.UserTableName).
    Where(filters).Build()

err := postgreSqlService.SelectOne(&user, ctx, query, args...)
```

### Multiple Rows
```go
var users []model.User
err := postgreSqlService.SelectMany(&users, ctx, query, args...)
```

### Count
```go
count, err := postgreSqlService.CountWithFilter(ctx, db.UserTableName, sql_query.WhereQuery{
    "is_deleted": {Operator: sql_query.SQLOperatorEqual, Value: false},
})
```

## Insert

### Single Row (Struct-Based)
```go
insertedId, err := postgreSqlService.InsertOneWithData(ctx, db.UserTableName, userData)
```

### With Custom RETURNING
```go
result := &model.User{}
_, err := postgreSqlService.InsertOneWithData(ctx, db.UserTableName, userData, service.ReturningConfig{
    Column:      []string{"id", "first_name", "email"},
    Destination: result,
})
```

### Bulk Insert
```go
_, err := postgreSqlService.InsertManyWithData(ctx, db.UserTableName, []model.User{u1, u2, u3})
```

## Update

### Single Row (Map-Based)
```go
_, err := postgreSqlService.UpdateOneWithData(ctx, db.UserTableName,
    sql_query.WhereQuery{
        "id": {Operator: sql_query.SQLOperatorEqual, Value: userId},
    },
    sql_query.UpdateQuery{
        "first_name": "Jane",
        "last_name":  "Doe",
    },
)
```

### With RETURNING Data
```go
returningData := &model.User{}
_, err := postgreSqlService.UpdateOneWithData(ctx, db.UserTableName, filter, updateBody,
    service.ReturningConfig{
        Column:      []string{"id", "first_name", "last_name"},
        Destination: returningData,
    },
)
```

### Bulk Row-Level Update
```go
_, err := postgreSqlService.UpdateEachWithData(ctx, db.UserTableName, "id", filter, []model.User{u1, u2})
```

## Soft Delete

Tables must have `is_deleted` (bool) and `deleted_at` (timestamp) columns.

```go
// Single
_, err := postgreSqlService.SoftDeleteOne(ctx, db.UserTableName, filter)

// Multiple
count, err := postgreSqlService.SoftDeleteMany(ctx, db.UserTableName, filter)

// Restore
_, err = postgreSqlService.RestoreSoftDeleteOne(ctx, db.UserTableName, filter)
count, err = postgreSqlService.RestoreSoftDeleteMany(ctx, db.UserTableName, filter)
```

## Hard Delete

```go
_, err := postgreSqlService.DeleteOneWithFilter(ctx, db.UserTableName, filter)
count, err := postgreSqlService.DeleteManyWithFilter(ctx, db.UserTableName, filter)
```

## Transactions

```go
tx, _ := postgreSqlService.GetPool().Begin(ctx)
postgreSqlService.SetTransaction(tx)

// ... execute queries within the transaction ...

err := postgreSqlService.CommitTransaction(ctx)
if err != nil {
    postgreSqlService.RollbackTransaction(ctx)
}
```

## Debug Logging

Enable SQL query logging during development. **Remove in production.**

```go
postgreSqlService.Debug()    // Level 1: log queries
postgreSqlService.Debug(2)   // Level 2: log queries + args (separated)
postgreSqlService.Debug(3)   // Level 3: log queries + args (combined)
```
