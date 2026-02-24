# Model and Constant Layouts

At the root of the project structure, you will find `model/` and `constant/`. These two directories house system-wide shared data shapes and immutable declarations.

## `/model`
The `model/` directory **strictly defines Database Entities**. 
- Structs inside `/model` map 1:1 with tables in your database (e.g. Postgres, MongoDB schemas). 
- These structs utilize `db:""` and `json:""` tag mappings.
- **Do not** put HTTP Request inputs or Response payloads (DTOs) here. Those belong isolated inside the respective `internal/api/{domain}/dto` folder. Model structs exist primarily so Activities can marshal and unmarshal raw rows from the database.

## `/constant`
The `constant/` directory defines system-wide **immutable declarations and configurations**.
- Store Temporal Workflow name registrations (e.g., `const WorkflowCreateUser = "CreateUserWorkflow"`).
- Store specific known error strings (e.g., `const ErrEmailNotRegistered = "Email not registered"`).
- Organize logically into files like `activity.go`, `workflow.go`, and `header.go`.
