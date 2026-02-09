<!--
SYNC IMPACT REPORT
==================
Version change: 1.0.0 → 1.1.0

Modified principles: None (all Phase II principles unchanged)

Added sections:
  - Phase III: Todo AI Chatbot (Project scope extension)
  - Principle VI: Backward Compatibility (Phase III-specific)
  - Principle VII: Stateless Architecture (Phase III-specific)
  - Principle VIII: AI-First Design (Phase III-specific)
  - Principle IX: Tool-Centric Orchestration (Phase III-specific)
  - Phase III Key Standards (Architecture, API, Database, Security, Code Quality)
  - Phase III Constraints (Technology, Endpoint Isolation, Environment)
  - Phase III Success Criteria (Functional, Architecture, Security, Quality)

Removed sections: None

Templates requiring updates:
  ✅ .specify/templates/plan-template.md - "Constitution Check" section present,
     compatible; Phase III plans MUST check both Phase II and III principles
  ✅ .specify/templates/spec-template.md - Requirements/Success Criteria aligned;
     Phase III specs MUST reference backward compatibility and stateless design
  ✅ .specify/templates/tasks-template.md - Phase structure compatible; Phase III
     tasks MUST include MCP tool, agent, and conversation persistence phases

Follow-up TODOs: None
-->

# Todo Full-Stack Web Application Constitution

## Project

**Todo Full-Stack Web Application (AI-Driven Development)**

## Core Principles

### I. Security First

All user data and API access MUST be protected through JWT authentication and strict authorization.

- All endpoints MUST require valid JWT tokens
- User ID MUST be validated against token claims for every request
- No unauthenticated access is permitted
- Shared secret MUST be stored via `BETTER_AUTH_SECRET` environment variable
- Token expiry MUST be enabled with maximum 7-day validity

### II. Spec-Driven Development

Every feature MUST originate from a written specification before implementation.

- No code may be written without a corresponding spec document
- Specifications MUST be reviewed and approved before implementation begins
- Changes to implemented features MUST be reflected in updated specifications
- All iterations MUST be documented in the spec workflow chain

### III. Automation Over Manual Coding

All development MUST follow the Agentic Dev Stack workflow using Claude Code and Spec-Kit Plus.

- Mandatory process:
  1. Write Specification (`/sp.specify`)
  2. Generate Implementation Plan (`/sp.plan`)
  3. Break into Tasks (`/sp.tasks`)
  4. Implement via Claude Code (`/sp.implement`)
  5. Review, Test, and Iterate
- No direct manual coding is permitted outside the agentic workflow
- All changes MUST be traceable to a task in `tasks.md`

### IV. Reliability & Consistency

Backend, frontend, and database systems MUST behave consistently under load and failure conditions.

- API responses MUST use proper HTTP status codes
- Error messages MUST be clear and consistent
- All operations MUST enforce ownership validation
- Systems MUST gracefully handle edge cases and failure scenarios

### V. Scalability by Design

Architecture MUST support multiple concurrent users and future feature expansion.

- Database queries MUST be optimized for concurrent access
- API design MUST follow RESTful standards
- Components MUST be modular to allow independent scaling
- Performance budgets MUST be defined and monitored

---

## Phase III: Todo AI Chatbot

> The following principles (VI–IX) apply to Phase III functionality.
> They extend—and MUST NOT contradict—Principles I–V above.

### VI. Backward Compatibility

All Phase II functionality MUST be preserved without modification when
implementing Phase III features.

- Phase II endpoints (`/api/tasks`, `/api/auth/*`) MUST remain unchanged
- Phase II database tables (User, Task) MUST NOT be altered in schema
- Phase III MUST introduce only new, isolated endpoints and tables
- Existing Phase II tests MUST continue to pass after Phase III deployment
- Shared infrastructure (database connection, auth middleware) MAY be
  reused but MUST NOT change behavior for Phase II consumers

### VII. Stateless Architecture

Every Phase III request MUST be independent; no in-memory session state
is permitted.

- The `/api/{user_id}/chat` endpoint MUST reconstruct conversation
  context from the database on every request
- No request-scoped data MAY persist in server memory between calls
- Conversation history MUST be stored in the Conversation and Message
  database tables
- Backend instances MUST be horizontally scalable without session
  affinity
- Database transactions MUST ensure consistency for concurrent requests

### VIII. AI-First Design

All Phase III task management flows MUST be driven through the OpenAI
Agents SDK; no direct CRUD bypasses are permitted from the chat
interface.

- User intent MUST be interpreted by the OpenAI Agents SDK before any
  task operation occurs
- MCP tools MUST be the sole mechanism for task mutations from the chat
  endpoint
- AI responses MUST be natural language; raw database results MUST NOT
  be exposed to users
- Agent errors MUST be caught and translated to user-friendly messages
- Tool descriptions MUST be clear and unambiguous to enable accurate
  intent classification

### IX. Tool-Centric Orchestration

All MCP tools MUST be invoked exclusively via the Backend Agent; no
separate MCP Agent process is required.

- The MCP SDK MUST run in-process within the FastAPI backend
- Every MCP tool MUST validate `user_id` ownership before operating on
  task records
- Tool parameters MUST be validated via Pydantic models
- Tool results MUST be JSON-serializable
- Tool invocations (name, arguments, result) MUST be persisted in the
  Message table for auditability

---

## Key Standards

### Architecture

| Layer | Technology |
|-------|------------|
| Frontend | Next.js 16+ (App Router) |
| Backend | FastAPI (Python) |
| ORM | SQLModel |
| Database | Neon Serverless PostgreSQL |
| Authentication | Better Auth with JWT |

### Security

- All endpoints MUST require valid JWT tokens
- Shared secret MUST be stored via `BETTER_AUTH_SECRET`
- Token expiry MUST be enabled (maximum 7 days)
- User ID MUST be validated against token claims
- No unauthenticated access is permitted

### API Design

- RESTful standards MUST be followed
- Proper HTTP status codes MUST be used
- Error messages MUST be clear and consistent
- Ownership enforcement MUST be applied on all operations

### Code Quality

- Project structure MUST be modular
- Type safety MUST be enforced
- Naming conventions MUST be consistent
- Linting and formatting MUST be applied
- Code MUST be documented

### Development Workflow

1. Write Specification
2. Generate Implementation Plan
3. Break into Tasks
4. Implement via Claude Code
5. Review, Test, and Iterate

> No direct manual coding is permitted outside the agentic workflow.

---

### Phase III Key Standards

#### Architecture (Phase III)

| Layer | Technology |
|-------|------------|
| Frontend | OpenAI ChatKit |
| Backend | FastAPI (Python) — extended |
| AI Framework | OpenAI Agents SDK |
| MCP Server | Official MCP SDK (in-process) |
| ORM | SQLModel |
| Database | Neon Serverless PostgreSQL |
| Authentication | Better Auth with JWT |

#### API Design (Phase III)

- Phase II endpoints MUST remain unchanged
- Phase III endpoint: `POST /api/{user_id}/chat` (stateless)
- `{user_id}` path parameter MUST be validated against JWT token claims
- Request body MUST include `message` (string) and optional
  `conversation_id` (UUID)
- Response body MUST include `response` (string) and
  `conversation_id` (UUID)

#### Database (Phase III)

- New tables: `Conversation`, `Message`
- Phase II tables (`User`, `Task`) MUST NOT be modified
- All tables MUST use UUIDs for primary keys
- All tables MUST include `created_at` and `updated_at` timestamps
- Foreign key constraints MUST be enforced
  (`Conversation.user_id → User.id`,
   `Message.conversation_id → Conversation.id`)
- MCP tools MAY read/write existing `Task` records but MUST NOT alter
  the Task schema

#### Security (Phase III)

- JWT validation MUST occur on every `/api/{user_id}/chat` request
- MCP tools MUST validate `user_id` ownership before every operation
- No hardcoded secrets; all credentials MUST use `.env` files
- Phase III environment variables (`OPENAI_API_KEY`,
  `NEXT_PUBLIC_OPENAI_DOMAIN_KEY`, `AUTH_SECRET`, `DATABASE_URL`)
  MUST NOT conflict with Phase II variables
- Internal agent/tool errors MUST NOT be exposed to users

#### Code Quality (Phase III)

- Pydantic validation MUST be applied on all request/response models
- Tool results MUST be JSON-serializable
- Conversation persistence MUST use database transactions
- Both user messages and assistant responses MUST be persisted
- Tool invocations MUST be logged in the Message table

---

## Constraints

### Technology

- Authentication: Better Auth + JWT
- Database: Neon PostgreSQL
- ORM: SQLModel
- Backend: FastAPI
- Frontend: Next.js (App Router)

### Security & Configuration

- All secrets MUST be stored in environment variables
- No hardcoded credentials are permitted
- Production secrets MUST be isolated from development

### Performance

- Average API response time MUST be < 300 ms
- Frontend initial load time MUST be < 2 seconds
- Database queries MUST be optimized

### Compatibility

- Latest stable versions only
- Cross-browser support required:
  - Chrome
  - Firefox
  - Edge
  - Safari

### Phase III Constraints

#### Technology (Phase III)

- AI Framework: OpenAI Agents SDK
- MCP Server: Official MCP SDK (in-process, no separate agent)
- Chat Frontend: OpenAI ChatKit
- All Phase II technology constraints continue to apply

#### Endpoint Isolation

- Phase II endpoints (`/api/tasks`, `/api/auth/*`) MUST NOT be modified
- Phase III adds only `POST /api/{user_id}/chat`
- No shared in-memory state between Phase II and Phase III endpoints

#### Environment Isolation

- Phase III MUST use dedicated environment variables where applicable
- Phase III variables MUST NOT override or shadow Phase II variables
- ChatKit frontend deployed on Vercel; backend on Hugging Face

---

## Success Criteria

### Functional

- All 5 basic-level features implemented
- All API endpoints operational
- Multi-user support verified
- Task ownership enforced

### Security

- 100% authenticated endpoints
- No cross-user data leakage
- JWT validation verified
- Security testing passed

### Workflow

- Complete spec → plan → task → implementation chain
- All iterations documented
- No manual code violations

### Quality

- All automated tests passing
- Zero critical bugs
- Clean deployment pipeline
- Production-ready build

### Review & Acceptance

- Meets Spec-Kit Plus requirements
- Passes technical evaluation
- Approved by project reviewers
- Ready for production deployment

### Phase III Success Criteria

#### Functional (Phase III)

- Users can manage tasks via natural language through ChatKit
- All 5 MCP tools operational (add_task, list_tasks, complete_task,
  delete_task, update_task)
- Conversation history persisted and retrievable across sessions
- Phase II functionality fully preserved and operational

#### Architecture (Phase III)

- Stateless chat endpoint handles concurrent users without session
  affinity
- MCP tools run in-process via Backend Agent (no separate MCP process)
- Conversation context reconstructed from database on every request

#### Security (Phase III)

- JWT validation on every chat request
- `user_id` ownership enforced on all MCP tool operations
- No internal agent/tool errors exposed to users
- Environment variables isolated from Phase II

#### Quality (Phase III)

- All Phase II tests continue to pass
- Phase III endpoints respond within performance budgets
- Conversation persistence verified with concurrent requests
- ChatKit frontend renders correctly across supported browsers

---

## Governance

This constitution supersedes all other development practices for this project.

**Amendment Process:**
1. Proposed changes MUST be documented with rationale
2. Changes MUST be reviewed for impact on existing artifacts
3. Version number MUST be incremented according to semantic versioning:
   - MAJOR: Backward-incompatible principle changes or removals
   - MINOR: New principles or sections added
   - PATCH: Clarifications or wording refinements
4. All dependent templates MUST be updated for consistency

**Compliance:**
- All PRs and code reviews MUST verify constitution compliance
- Violations MUST be documented and justified in the Complexity
  Tracking section of plan.md
- Runtime guidance is maintained in CLAUDE.md
- Phase III changes MUST additionally verify backward compatibility
  with Phase II (Principle VI)

**Version**: 1.1.0 | **Ratified**: 2026-02-06 | **Last Amended**: 2026-02-09
