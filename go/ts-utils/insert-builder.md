# Insert Builder

Use `NewSQLInsertBuilder[T]("table")` to construct INSERT queries. Column names are extracted from struct tags (`column` or `json`), and Snowflake IDs are auto-generated.

## Single Insert

Pass a struct to `.Insert()`. By default it returns the inserted `id::text`.

```go
query, args, _ := sql_query.
    NewSQLInsertBuilder[model.User](db.UserTableName).
    Insert(model.User{
        FirstName: "John",
        LastName:  "Doe",
        Email:     "john@example.com",
    }).
    Build()
```

## Bulk Insert

Pass a slice of structs for multi-row inserts. All rows must have the same set of columns.

```go
query, args, _ := sql_query.
    NewSQLInsertBuilder[model.User](db.UserTableName).
    Insert([]model.User{user1, user2, user3}).
    Build()
```

## Custom RETURNING Columns

```go
.Insert(user, "id", "created_at")
```

## Exclude Empty Fields

Skip zero-value fields when inserting a single struct.

```go
.Insert(user).ExcludeEmpty()
```

> This has no effect on bulk inserts, because all rows must share the same column set.

## Upsert (ON CONFLICT)

```go
.Insert(user).Conflict("(email)", "NOTHING")
// → INSERT ... ON CONFLICT (email) DO NOTHING

.Insert(user).Conflict("(id)", "UPDATE SET name = EXCLUDED.name")
// → INSERT ... ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name
```
