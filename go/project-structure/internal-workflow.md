# Internal Workflow Layout (`/internal/workflow`)

The `internal/workflow` directory is the beating heart of your application's **Business Logic and Orchestration**. 

It dictates *what* happens and in *what order*, handling compensations (rollbacks) and coordinating multiple activities. However, it cannot perform native side-effects directly (like executing SQL or making HTTP client calls).

## Directory Structure
Like the API layer, workflows must be organized strictly by **Domain Context**.

### Example Layout
```text
internal/
└── workflow/
    ├── workflow.go             // Main registry for all application workflows
    ├── user/                   // Domain: User
    │   ├── create_user_workflow.go
    │   ├── create_user_workflow_test.go
    │   ├── edit_user_workflow.go
    │   └── edit_user_workflow_test.go
    └── position/               // Domain: Position
        ├── create_position_workflow.go
        └── create_position_workflow_test.go
```

## Rules for Workflows
1. **No External Imports:** Workflows must remain entirely deterministic. They cannot import `database/sql`, `net/http`, or use Go's native `time.Now()` (must use `workflow.Now()`).
2. **One File per Workflow:** Define only one main workflow function per file to keep orchestration logic highly readable.
3. **Collocated Tests:** Unit tests for a workflow (`_test.go`) must live exactly beside the workflow implementation file.
