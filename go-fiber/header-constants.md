# Header Constants

Use immutable package constants when referencing HTTP headers, context variable keys, or environment bindings instead of rewriting string literals.

```go
// Bad:
// Repeating hardcoded strings raises the risk of typos across files.
func GetGlobalUserList(ctx *fiber.Ctx) error {
	clientId := ctx.Get("X-Client-Id")
	loggedInUserId := ctx.Get("X-User-Id")

    // ...
}

// Good:
func GetGlobalUserList(ctx *fiber.Ctx) error {
	clientId := ctx.Get(util_constant.HEADER_CLIENT_ID)
	loggedInUserId := ctx.Get(util_constant.HEADER_USER_ID)

    // ...
}
```

This acts as a single source of truth; if the underlying header key standard changes throughout the ecosystem, updating one constant scales effortlessly.
