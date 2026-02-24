# Internal Activity Layout (`/internal/activity`)

The `internal/activity` directory is the **Side-Effect Execution Layer**. 

Activities are the physical workers that execute the commands issued by workflows. This is the **only** layer in the `internal/` structure permitted to directly connect to databases, call external third-party HTTP services, or write files to a disk.

## Directory Structure
Activities are organized by **Domain Context**, maintaining symmetry with the `api` and `workflow` directories.

### Example Layout
```text
internal/
└── activity/
    ├── activity.go             // Defines the master Activities struct holding DB/Config dependencies
    ├── auth/                   // Domain: Auth
    │   └── check_token_activity.go
    └── user/                   // Domain: User
        ├── create_user_activity.go
        ├── delete_user_activity.go
        └── get_user_detail_activity.go
```

## Rules for Activities
1. **Isolate Dependencies:** Activities rely on the dependency-injected `service.Managerer` to pull Database Providers or external clients.
2. **Keep it Dumb:** Activities should do exactly one thing (e.g., "Insert this User struct into Postgres"). They should not contain complex branching business logic; leave orchestrating multiple inserts to the Workflow.
3. **Format Errors Properly:** If a database query yields `pgx.ErrNoRows`, the Activity must wrap it in `entity.WorkflowNotFound` before returning to prevent upstream 500 server errors.
