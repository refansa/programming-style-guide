# Select Builder

Use `NewSQLSelectBuilder[T]("table")` to construct type-safe SELECT queries. The generic type `T` auto-extracts JSON tags as default column names.

## Basic Usage

```go
query, args, _ := sql_query.
    NewSQLSelectBuilder[model.User](db.UserTableName).
    Where(sql_query.WhereQuery{
        "id": {Operator: sql_query.SQLOperatorEqual, Value: userId},
    }).
    Build()

var user model.User
err := dbService.SelectOne(&user, ctx, query, args...)
```

## Aliased Tables

```go
sql_query.NewSQLSelectBuilder[model.User](db.UserTableName, "u").
    Select("u.id", "u.name")
```

## Joins

```go
builder.
    LeftJoin("divisions d", "d.id = u.division_id").
    Join("positions p", "p.id = u.position_id")
```

## Pagination

Pass the `Pagination` struct directly from a parsed query string to handle LIMIT, OFFSET, and ORDER BY automatically.

```go
builder.Paginate(sql_query.Pagination{
    Page:      query.Page,
    Limit:     query.Limit,
    SortBy:    query.SortBy,
    SortOrder: query.SortOrder,
})
```

## Search

Case-insensitive ILIKE search across multiple fields combined with OR. Supports array columns via the `:array` suffix.

```go
builder.Search(query.Keyword, []string{"first_name", "last_name", "tags:array"})
```

## Count Queries

Use `NewSQLCountBuilder` for queries that only need the total row count.

```go
query, args, _ := sql_query.
    NewSQLCountBuilder(db.UserTableName).
    Where(filters).
    Build()
```

## JSON Aggregation

Build nested JSON objects or arrays directly in SQL. The `dto` parameter maps JSON keys to SQL column expressions.

```go
builder.SelectJSONAggregate(
    "division",                    // alias
    model.DivisionDTO{},           // struct with json tags
    "d.id IS NOT NULL",            // condition
    false,                         // asArrayAggregation
)
```

## Common Table Expressions (CTE)

```go
cte := sql_query.NewSQLSelectBuilder[model.Order]("orders").
    Select("id", "user_id").
    Where(filters)

mainQuery := sql_query.NewSQLSelectBuilder[model.User]("users").
    WithCTEBuilder("recent_orders", cte.(*sql_query.SelectBuilder).SQLEloquentQuery).
    Join("recent_orders ro", "ro.user_id = users.id")
```

## Build

Always call `.Build()` at the end. It returns `(queryString, args, error)`.

```go
query, args, err := builder.Build()
```
