# Internal API Layout (`/internal/api`)

The `internal/api` directory is the **strict HTTP boundary** of the application. Business logic, orchestrations, and direct database queries do not belong here. 

Code in this directory is responsible solely for:
1. Parsing incoming HTTP requests.
2. Validating input shapes.
3. Triggering the underlying business orchestrator (Temporal).
4. Formatting the response payload back to the client.

## Directory Structure
The `api` folder must be organized by **Domain Context** (e.g. `user`, `position`, `auth`), not by technical function. 

Inside each domain context, maintain the following 3 sub-packages:

- **`/controller`**: Contains the Fiber handler functions. 
- **`/dto`**: Defines the Request (Body/Query) and Response struct shapes.
- **`/route`**: Handles mounting the controllers to `fiber.Router` groups with appropriate middleware.

### Example Layout
```text
internal/
└── api/
    ├── route.go              // Main router that mounts all domains
    ├── user/                 // Domain: User
    │   ├── controller/
    │   │   ├── create_user_controller.go
    │   │   └── get_user_controller.go
    │   ├── dto/
    │   │   ├── create_user_dto.go
    │   │   └── user_response_dto.go
    │   └── route/
    │       └── route.go      // Mounts user controllers to the /users path
    └── position/             // Domain: Position
        ├── controller/
        ├── dto/
        └── route/
```

By organizing endpoints by domain, engineers can quickly locate all HTTP surface area related to a specific feature without jumping across massive, monolithic `controllers/` or `routes/` directories.
