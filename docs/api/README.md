# API Documentation

API contracts for the management API (`/api/v1`) and agent-facing endpoints.

Planned contents:

- OpenAPI specification for management APIs.
- Agent enrollment, heartbeat, telemetry and policy endpoints.
- Response-action contract with authorization, approval, expiry and result fields.
- Authentication, scoping and error-code conventions.

Contracts are proposals until implemented and tested. All management APIs require
user authentication and server-side authorization; agent APIs require unique
agent identity and narrowly scoped permissions.
