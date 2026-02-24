# Go Project Structure (Domain-Driven Design)

The Go style guide enforces a structured, Domain-Driven Design (DDD) layout tailored specifically for building robust Temporal microservices over Fiber interfaces. This structure prevents spaghetti code by isolating HTTP routing, Orchestration, and Side-effects into distinct layers.

## Architecture Guidelines

- [Internal API Layout](internal-api.md) (`internal/api`)
- [Internal Workflow Layout](internal-workflow.md) (`internal/workflow`)
- [Internal Activity Layout](internal-activity.md) (`internal/activity`)
- [Internal App Layout](internal-app.md) (`internal/app`)
- [Model and Constants Layout](model-and-constant.md) (`model`, `constant`)
