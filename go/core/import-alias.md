# Import Aliasing

Import aliasing must be used if the package name does not match the last
element of the import path.

```go
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```

In all other scenarios, import aliases should be avoided unless there is a
direct conflict between imports.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"

  util_helper "gitea.tagsamurai.local/BETS-V2/ts-utils/v2/helper"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "gitea.tagsamurai.local/BETS-V2/iam-service/helper"
  util_helper "gitea.tagsamurai.local/BETS-V2/ts-utils/v2/helper"
)
```

</td></tr>
</tbody></table>
