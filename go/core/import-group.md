# Import Group Ordering

There should be two import groups:

- Standard library
- Everything else

This is the grouping applied by goimports by default.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  "gitea.tagsamurai.local/BETS-V2/ts-utils/..."
  "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "gitea.tagsamurai.local/BETS-V2/ts-utils/..."
  "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>
