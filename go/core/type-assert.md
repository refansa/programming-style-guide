# Handle Type Assertion Failures

The single return value form of a [type assertion] will panic on an incorrect
type. Therefore, always use the "comma ok" idiom.

  [type assertion]: https://go.dev/ref/spec#Type_assertions

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```

</td></tr>
</tbody></table>

There are a few situations where the single assignment form is fine, because we already knows what the type is, or we want the incorrect type to defaults to it's zero value. In such case, it is permissible to dismiss the "comma ok" idiom to avoid error.

```go
t, _ := i.(string)
```
