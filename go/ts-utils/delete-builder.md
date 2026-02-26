# Delete Builder

Use `NewSQLDeleteBuilder("table")` to construct DELETE queries. By default it returns the deleted `id`.

## Basic Delete

```go
query, args, _ := sql_query.
    NewSQLDeleteBuilder(db.UserTableName).
    Delete().
    Where(sql_query.WhereQuery{
        "id":         {Operator: sql_query.SQLOperatorEqual, Value: userId},
        "is_deleted": {Operator: sql_query.SQLOperatorEqual, Value: false},
    }).
    Build()
```

## Custom RETURNING Columns

```go
.Delete("id", "deleted_at")
// → DELETE FROM ... RETURNING id, deleted_at
```

## Multi-Table Delete (USING)

For deletes that need to reference another table in the WHERE clause.

```go
query, args, _ := sql_query.
    NewSQLDeleteBuilder(db.UserTableName).
    Delete().
    Using([]string{"roles"}).
    Where(sql_query.WhereQuery{
        "": {Operator: sql_query.SQLOperatorRaw, Value: `users.role_id = roles.id`},
    }).
    Build()
// → DELETE FROM users USING roles WHERE users.role_id = roles.id RETURNING id
```
