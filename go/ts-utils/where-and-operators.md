# WHERE Clauses and SQL Operators

All query builders share the same `WhereQuery` type and `SQLCondition` struct for building parameterized WHERE clauses.

## WhereQuery

`WhereQuery` is a `map[string]SQLCondition`. Each key is a column name and the value defines the operator and comparison.

```go
sql_query.WhereQuery{
    "id":         {Operator: sql_query.SQLOperatorEqual, Value: 1},
    "is_deleted": {Operator: sql_query.SQLOperatorEqual, Value: false},
}
// → "id" = $1 AND "is_deleted" = $2
```

## SQLCondition Fields

| Field          | Type           | Description                                               |
|----------------|----------------|-----------------------------------------------------------|
| `Operator`     | `SQLOperators` | The SQL comparison operator                               |
| `Value`        | `any`          | Literal value, slice, or subquery string                  |
| `IsRef`        | `bool`         | `true` = value is a column reference, not a literal       |
| `SourceIsValue`| `bool`         | `true` = key side is a literal, not a column              |
| `IsSubQuery`   | `bool`         | `true` = value is a raw SQL subquery string               |
| `IsEpochTime`  | `bool`         | `true` = value is epoch/unix milliseconds                 |
| `IsArray`      | `bool`         | `true` = column contains a JSON array of objects          |
| `Key`          | `string`       | JSON object key (used with `IsArray`)                     |
| `ExtraArgs`    | `[]any`        | Additional args for `SQLOperatorRaw`                      |

## SQL Operators

### Comparison
| Constant                  | SQL        | Example                                                     |
|---------------------------|------------|-------------------------------------------------------------|
| `SQLOperatorEqual`        | `=`        | `{"age": {Operator: SQLOperatorEqual, Value: 30}}`          |
| `SQLOperatorNotEqual`     | `!=`       | `{"status": {Operator: SQLOperatorNotEqual, Value: "old"}}` |
| `SQLOperatorGreaterThan`  | `>`        | `{"score": {Operator: SQLOperatorGreaterThan, Value: 90}}`  |
| `SQLOperatorLessThan`     | `<`        | `{"price": {Operator: SQLOperatorLessThan, Value: 100}}`    |
| `SQLOperatorGTE`          | `>=`       | `{"age": {Operator: SQLOperatorGTE, Value: 18}}`            |
| `SQLOperatorLTE`          | `<=`       | `{"age": {Operator: SQLOperatorLTE, Value: 65}}`            |

### Set
| Constant              | SQL        | Example                                                       |
|-----------------------|------------|---------------------------------------------------------------|
| `SQLOperatorIn`       | `IN`       | `{"id": {Operator: SQLOperatorIn, Value: []int{1,2,3}}}`     |
| `SQLOperatorNotIn`    | `NOT IN`   | `{"id": {Operator: SQLOperatorNotIn, Value: []int{1,2,3}}}`  |
| `SQLOperatorAny`      | `ANY`      | `{"tags": {Operator: SQLOperatorAny, Value: pq.Array(arr)}}` |

### Pattern Matching
| Constant               | SQL          | Example                                                      |
|------------------------|--------------|--------------------------------------------------------------|
| `SQLOperatorLike`      | `LIKE`       | `{"title": {Operator: SQLOperatorLike, Value: "%hello%"}}`   |
| `SQLOperatorILike`     | `ILIKE`      | `{"name": {Operator: SQLOperatorILike, Value: "john%"}}`     |
| `SQLOperatorNotLike`   | `NOT LIKE`   | `{"title": {Operator: SQLOperatorNotLike, Value: "%test%"}}` |
| `SQLOperatorNotILike`  | `NOT ILIKE`  | `{"name": {Operator: SQLOperatorNotILike, Value: "doe%"}}`   |

### Null Checks
| Constant                | SQL            | Example                                                  |
|-------------------------|----------------|----------------------------------------------------------|
| `SQLOperatorIsNull`     | `IS NULL`      | `{"deleted_at": {Operator: SQLOperatorIsNull}}`          |
| `SQLOperatorIsNotNull`  | `IS NOT NULL`  | `{"deleted_at": {Operator: SQLOperatorIsNotNull}}`       |

### Range
| Constant                 | SQL            | Example                                                                               |
|--------------------------|----------------|---------------------------------------------------------------------------------------|
| `SQLOperatorBetween`     | `BETWEEN`      | `{"created_at": {Operator: SQLOperatorBetween, Value: []int64{start, end}, IsEpochTime: true}}` |
| `SQLOperatorNotBetween`  | `NOT BETWEEN`  | `{"price": {Operator: SQLOperatorNotBetween, Value: []int{10, 50}}}`                  |

### Subqueries / Existence
| Constant                | SQL            | Example                                                                       |
|-------------------------|----------------|-------------------------------------------------------------------------------|
| `SQLOperatorExist`      | `EXISTS`       | `{"": {Operator: SQLOperatorExist, Value: "(SELECT 1 ...)", IsSubQuery: true}}`      |
| `SQLOperatorNotExist`   | `NOT EXISTS`   | `{"": {Operator: SQLOperatorNotExist, Value: "(SELECT 1 ...)", IsSubQuery: true}}`   |

### Raw SQL
| Constant           | SQL       | Example                                                                              |
|--------------------|-----------|--------------------------------------------------------------------------------------|
| `SQLOperatorRaw`   | (custom)  | `{"": {Operator: SQLOperatorRaw, Value: "age > ? OR status = 'active'", ExtraArgs: []any{30}}}` |

## OR Conditions (WhereOr)

Use `.WhereOr()` to combine multiple filter groups with OR logic. Each group is AND-combined internally.

```go
builder.WhereOr(
    sql_query.WhereQuery{"role": {Operator: sql_query.SQLOperatorEqual, Value: "admin"}},
    sql_query.WhereQuery{"role": {Operator: sql_query.SQLOperatorEqual, Value: "editor"}},
)
// → (("role" = $1) OR ("role" = $2))
```
