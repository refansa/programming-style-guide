# Update Builder

Use `NewSQLUpdateBuilder("table")` to construct UPDATE queries. The builder automatically appends `updated_at = NOW()` unless explicitly skipped.

## Map-Based Update

Use `UpdateQuery` (a `map[string]any`) to define the SET clause. Keys are column names, values are the new data.

```go
query, args, _ := sql_query.
    NewSQLUpdateBuilder(db.UserTableName).
    Update(sql_query.UpdateQuery{
        "first_name": "Jane",
        "last_name":  "Doe",
        "email":      "jane@example.com",
    }).
    Where(sql_query.WhereQuery{
        "id": {Operator: sql_query.SQLOperatorEqual, Value: userId},
    }).
    Build()
```

## Raw SQL Expressions

For values that contain SQL expressions (not literals), use `UpdateRawSQL`:

```go
sql_query.UpdateQuery{
    "view_count": sql_query.UpdateRawSQL{
        Expr: `"view_count" + $?`,
        Args: []any{5},
    },
}
```

## Batch Update (UpdateEach)

Update multiple rows at once using a `FROM (VALUES ...)` pattern. Rows are matched by the given identifier column.

```go
query, args, _ := sql_query.
    NewSQLUpdateBuilder(db.UserTableName).
    UpdateEach([]model.User{user1, user2}, "id").
    Build()
// → UPDATE users SET name = v.name, updated_at = NOW()
//   FROM (VALUES ($1,$2),($3,$4)) AS v(id,name)
//   WHERE users.id = v.id
```

## Atomic Increment

Increment integer columns by a given value.

```go
query, args, _ := sql_query.
    NewSQLUpdateBuilder(db.UserTableName).
    Increment(map[string]any{"login_count": 1}).
    Where(filters).
    Build()
// → UPDATE users SET "login_count" = "login_count" + $1, "updated_at" = NOW()
```

## Conditional CASE Update (AddCase)

Build conditional SET clauses using CASE WHEN expressions.

```go
query, args, _ := sql_query.
    NewSQLUpdateBuilder(db.UserTableName).
    AddCase("status", func(b sql_query.UpdateCases) {
        b.Case(sql_query.MultiFilterCondition{
            And: sql_query.WhereQuery{
                "role": {Operator: sql_query.SQLOperatorEqual, Value: "admin"},
            },
        }, "Active", false)
        b.Else("status", true) // fallback to original column value
    }).
    Where(filters).
    Build()
```
