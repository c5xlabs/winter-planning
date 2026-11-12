---
name: mock-forge
external: github:paul-gross/blizzard#2180
code:
  - blizzard-mock/src/blizzard_mock/forge/api/middleware.py::LeverMiddleware
  - blizzard-mock/src/blizzard_mock/forge/domain/levers.py::Lever
  - blizzard-mock/src/blizzard_mock/forge/app.py::create_app
---

# Mock forge tokens and Apps

Lets the mock forge tell tokens apart and serve GitHub App installations, every addition off by default. Builds
[mock-forge.md](../spec/mock-forge.md). Its fine-grained row follows `decision:advanced-forge-auth/fine-grained-checks`.

## Change

- `STRICT_TOKENS`, the token and App levers, and their answers; `X-GitHub-Request-Id` on every answer.
- `MockApp`, `MockInstallation`, and minted tokens; `GET /app/installations/{id}` and
  `POST
  /app/installations/{id}/access_tokens`.

## Acceptance

- With every lever off, the mock forge's existing tests pass unchanged.
- Under `STRICT_TOKENS`: an unregistered or revoked token answers 401; a classic token reaches every route with its
  scopes echoed; a fine-grained token answers 403 on a private repository's check runs while the decision withholds
  them.
- A JWT signed with another key, an expired one, or one whose `exp` exceeds ten minutes answers 401; a mint narrows to
  the permissions asked for and refuses 422 one not granted.
- A minted token answers 401 after its lifetime, and 403 with `X-Accepted-GitHub-Permissions` on a route its permissions
  do not cover.
