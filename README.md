# lost-last

`lost-last` is a backend for a team-based CTF platform. It serves the participant-facing API, administrative challenge management, score calculation, flag submission, dynamic challenge instances, and the integration layer for `kctf`-backed web tasks.

The project is written in Go, uses Hertz for HTTP transport, PostgreSQL for persistent state, Redis for caching, and a separate gRPC flag checker service for validation and scoring workflows.

## What the project does

The backend is responsible for the core CTF lifecycle:

- registration, login, session cookies, and protected routes
- team management
- challenge catalog for participants
- flag submission and solve recording
- leaderboard and personal scoreboard
- dynamic challenge instances for teams
- administrative CRUD for challenges
- administrative deploy and stop flow for `kctf`-managed web tasks

In practice this means organizers can manage tasks and expose dynamic web challenges, while participants interact only with the platform API and challenge endpoints, not with the infrastructure underneath.

## Architecture

The repository contains two main runtime components:

| Component | Role |
| --- | --- |
| `main.go` | Main HTTP API for participants and admins |
| `cmd/flagchecker` | Separate gRPC flag checker service |

Supporting infrastructure:

| Service | Purpose |
| --- | --- |
| PostgreSQL | users, teams, challenges, solves, admin challenge metadata, instances |
| Redis | challenge and scoreboard cache |
| gRPC flag checker | flag validation and solve recording |
| `kctf` adapter | deploy/stop/status/access/logs bridge for dynamic web tasks |

The `kctf` codebase is vendored into [kctf](/Users/ivankor/ctf_platform/lost-last/kctf). The HTTP layer talks to an internal `kctf.Client` interface, so the API surface is already isolated from the actual cluster implementation.

## API surface

The project currently exposes the following HTTP routes.

### Public and participant routes

```text
GET    /ping

POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout

GET    /api/v1/me
PUT    /api/v1/me

POST   /api/v1/team
GET    /api/v1/team
DELETE /api/v1/team
POST   /api/v1/team/join

GET    /api/v1/teams
GET    /api/v1/teams/:team_id

GET    /api/v1/catalog/challenges
GET    /api/v1/catalog/challenges/:challenge_id
POST   /api/v1/catalog/challenges/:challenge_id/instance
GET    /api/v1/catalog/challenges/:challenge_id/instance
DELETE /api/v1/catalog/challenges/:challenge_id/instance
POST   /api/v1/catalog/challenges/:challenge_id/submit

GET    /api/v1/leaderboard
GET    /api/v1/scoreboard
```

### Admin challenge routes

Administrative challenge management is currently mounted under `/api/v1/admin/challenges` in [router.go](/Users/ivankor/ctf_platform/lost-last/router.go).

```text
GET    /api/v1/admin/challenges
POST   /api/v1/admin/challenges
GET    /api/v1/admin/challenges/:name
PUT    /api/v1/admin/challenges/:name
DELETE /api/v1/admin/challenges/:name
GET    /api/v1/admin/challenges/:name/status
GET    /api/v1/admin/challenges/:name/access
GET    /api/v1/admin/challenges/:name/logs
POST   /api/v1/admin/challenges/:name/deploy
POST   /api/v1/admin/challenges/:name/stop
```

If you want these admin routes to live at `/api/v1/challenges/...` without the `/admin` prefix, that requires changing the route registration in code, not just the documentation.

## kCTF integration

The project is prepared to manage `kctf`-backed web challenges through an internal adapter layer in [internal/kctf](/Users/ivankor/ctf_platform/lost-last/internal/kctf).

Right now the wiring in [main.go](/Users/ivankor/ctf_platform/lost-last/main.go) uses a stub client:

- the admin API is already stable
- challenge deploy/stop/status/access/logs flow is already modeled
- the real cluster adapter can replace the stub without changing handlers or routes

This is the intended control-plane split:

- backend owns users, teams, metadata, scoring, permissions, and challenge lifecycle
- `kctf` owns safe runtime isolation for deployable web tasks

## Local development

### Recommended way

The fastest way to run the stack locally is Docker Compose:

```bash
docker compose up --build
```

This starts:

- API on `:8080`
- PostgreSQL on `:5432` inside compose, mapped to `localhost:5433`
- Redis
- flag checker gRPC service

Health check:

```bash
curl http://localhost:8080/ping
```

### Running services manually

Main API:

```bash
go run .
```

Flag checker:

```bash
go run ./cmd/flagchecker
```

The application creates and migrates the database on startup through [internal/app/db.go](/Users/ivankor/ctf_platform/lost-last/internal/app/db.go).

## Configuration

Configuration is environment-driven and loaded in [internal/config/config.go](/Users/ivankor/ctf_platform/lost-last/internal/config/config.go).

Important variables:

| Variable | Purpose |
| --- | --- |
| `HTTP_ADDR` | API listen address |
| `DATABASE_URL` | PostgreSQL DSN |
| `REDIS_ADDR` | Redis address |
| `AUTH_JWT_SECRET` | JWT signing secret |
| `AUTH_ACCESS_TTL` | access token TTL |
| `CORS_ALLOWED_ORIGINS` | frontend origins |
| `FLAGCHECKER_GRPC_ADDR` | gRPC address of flag checker |
| `FLAGCHECKER_LISTEN_ADDR` | flag checker listen address |
| `FLAGCHECKER_HEALTH_ADDR` | flag checker health address |
| `KCTF_URL_SCHEME` | public scheme for `kctf` challenge URLs |
| `KCTF_PUBLIC_DOMAIN` | public domain for `kctf` challenge URLs |

For local work, the values in [compose.yaml](/Users/ivankor/ctf_platform/lost-last/compose.yaml) are the most useful reference.

## Project layout

| Path | Purpose |
| --- | --- |
| [biz/handler](/Users/ivankor/ctf_platform/lost-last/biz/handler) | HTTP handlers |
| [biz/router](/Users/ivankor/ctf_platform/lost-last/biz/router) | route registration |
| [idl](/Users/ivankor/ctf_platform/lost-last/idl) | protobuf and API contracts |
| [internal/app](/Users/ivankor/ctf_platform/lost-last/internal/app) | runtime wiring |
| [internal/service](/Users/ivankor/ctf_platform/lost-last/internal/service) | business logic |
| [internal/repo](/Users/ivankor/ctf_platform/lost-last/internal/repo) | persistence layer |
| [internal/cache](/Users/ivankor/ctf_platform/lost-last/internal/cache) | Redis-backed caches |
| [internal/kctf](/Users/ivankor/ctf_platform/lost-last/internal/kctf) | `kctf` integration boundary |
| [internal/scoring](/Users/ivankor/ctf_platform/lost-last/internal/scoring) | scoring logic |
| [cmd/flagchecker](/Users/ivankor/ctf_platform/lost-last/cmd/flagchecker) | gRPC flag checker binary |
| [kctf](/Users/ivankor/ctf_platform/lost-last/kctf) | vendored `kctf` sources |

## Testing and CI

Local test command:

```bash
go test ./...
```

The repository also contains a simple GitLab CI pipeline in [.gitlab-ci.yml](/Users/ivankor/ctf_platform/lost-last/.gitlab-ci.yml) that runs:

- `go test ./...`
- `go build ./...`

## Current status

The backend already covers the core platform flows and exposes an admin API for `kctf` challenge lifecycle management. The `kctf` integration point is in place, but the default runtime wiring still uses a stub client rather than a real Kubernetes-backed adapter.

That means the API contract is ready, the control plane shape is already defined, and the next infrastructure step is replacing the stub with a real cluster implementation.
