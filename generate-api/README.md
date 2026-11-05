# generate-api

Design, implement, refactor, and review production-ready REST APIs. The agent follows the project's existing language, framework, and layout, and applies the rules in [SKILL.md](SKILL.md) when those conventions are missing.

Use it when creating or changing endpoints, controllers, routes, handlers, services, or API contracts.

## What the agent follows

- Resource-oriented paths and HTTP method semantics, including safety and idempotency
- Status codes and one consistent error shape, preferring [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457) problem details
- Validation at the boundary, server-side authentication and authorization, and tenant isolation
- Filtering, sorting, search, and pagination
- Versioning that stays backward compatible until a real break needs a new version
- OpenAPI (or the project's existing docs) kept in sync with the implementation
- Functional, security, and HTTP-semantics tests

The skill is framework-agnostic. It does not add a new architecture or new libraries when the project already has a pattern.

## Install

Always install from GitHub, including while editing this skill. Push to `main` first.

Into the current project, for every detected agent (Cursor, Claude Code, OpenCode, Codex, and any others installed):

```bash
npx skills add bablukpik/skills --skill generate-api -y
```

Name them explicitly:

```bash
npx skills add bablukpik/skills --skill generate-api --agent cursor claude-code opencode codex -y
```

Other scopes are documented in [docs/installing.md](../docs/installing.md).

## Example requests

- "Add a REST API for orders with create, list, get, update, and cancel."
- "Review these endpoints for HTTP semantics, validation, and authorization."
- "Refactor `/getUsers` and `/createUser` into resource-oriented routes."
