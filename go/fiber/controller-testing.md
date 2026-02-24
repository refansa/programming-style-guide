# Controller Testing

When writing unit tests for Fiber controllers, test the HTTP layer holistically using `fiber.New()` and the `app.Test()` method alongside native `httptest` request builders.

```go
// Good:
func TestGetGlobalUserDetail(t *testing.T) {
	t.Run("Success_GetUsers", func(t *testing.T) {
		// 1. Setup mock managers and inject them into the controller struct
        // ... mockServiceMgr setup
		controller := &UserController{
			ServiceManager: mockServiceMgr,
		}

		// 2. Initialize an ephemeral fiber instance just for this test
		app := fiber.New()
		app.Get("/:id", controller.GetGlobalUserDetail)

		// 3. Craft a native Go HTTP request simulating the client
		req := httptest.NewRequest("GET", `/1`, nil)
		req.Header.Set(util_constant.HEADER_CLIENT_ID, "qtera")

		// 4. Inject the request through Fiber's HTTP pipeline internally
		resp, err := app.Test(req)

		// 5. Assert the Fiber response without needing network connectivity
		assert.NoError(t, err)
		assert.Equal(t, fiber.StatusOK, resp.StatusCode)

        // ... assert mock expectations
	})
}
```

This ensures your middleware, path parameter bindings (`/:id`), header parsers, and status code writes are tested as truthfully as possible without the overhead of binding and tearing down real local network listener ports.
