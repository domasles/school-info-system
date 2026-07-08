# SIS Planning Document

This document captures the complete planning conversation and decisions made for the School Information System (SIS) project.

## Project Vision

Build a modern, AI-powered school information system designed primarily for Lithuanian schools, with multi-language support for international expansion. The system should be ready-to-use out of the box, not a framework that schools need to build upon.

Key principle: Schools should be able to deploy and start using the system within 40 minutes, with all Lithuanian educational standards pre-configured.

---

## Role-Based Access Control (RBAC)

The following roles were defined based on Lithuanian school structures:

### Administratorius (Administrator)

- Manages the entire system: creates accounts, changes roles
- Views audit logs
- No rights to view or edit grades, attendance, assignments, schedules
- Messages: only system notifications (technical, alerts)

### Direktorius (Director)

- Visibility: everything, to understand what's happening in the school
- Editing: minimal - not necessarily editing grades or schedules
- Messages: can send to all school participants, receive from all

### Pavaduotojas (Deputy Director)

- Responsible for schedules and lesson distribution
- Can edit schedules, distributions, substitute lessons
- Visibility: mostly their own area, but not all school grades
- Messages: can send to all school participants, receive from all

### Specialistai (Specialists - psychologists, social pedagogues)

- Read-only access to grades, attendance, some notes
- No editing rights
- Messages: can send to all staff, receive from students, parents, other staff

### Mokytojas (Teacher)

- Can write grades, assignments, mark attendance only for their own subject classes
- Can send announcements to their students
- No access to whole school data
- Messages: can send to everyone (students, parents, staff), receive from all

### Klasės auklėtojas (Homeroom Teacher)

- All teacher permissions plus:
  - Excuse absences
  - View all students in their class
  - Send announcements to their class
  - Manage class profile
- Messages: can send to everyone, receive from all

### Tėvai (Parents)

- Read-only rights only for their own child:
  - Grades, attendance, assignments, announcements
- No rights to edit any pedagogical data
- Messages: can only send messages to school staff, receive responses from staff

### Mokiniai (Students)

- Read-only rights only for themselves:
  - Grades, attendance, assignments, announcements
- No editing rights
- Messages: can only send messages to school staff, receive responses from staff

---

## Architecture Decisions

### Comparison: Initial DDD Proposal vs EchoTuner-Inspired Approach

The initial proposal followed strict Domain-Driven Design with deeply nested structures:
- Entities, Value Objects, Aggregates per bounded context
- CQRS with Commands, Queries, Handlers
- Repository per domain

After analyzing the domasles/echotuner repository, a leaner approach was adopted:

| Aspect | Initial Proposal | Final Approach |
|--------|-----------------|----------------|
| Domain structure | Deeply nested DDD | Flat services per feature |
| Service pattern | Complex DI | SingletonServiceBase |
| Repository | One per context | One generic repository for all |
| Application layer | Use cases | Pydantic models as DTOs |
| Files in API | ~80+ | ~45 |
| Nesting depth | 5-6 levels | 3-4 levels |

### Key Patterns from EchoTuner

1. **SingletonServiceBase**: All services extend this base class with managed lifecycle
2. **Generic Repository**: Single repository.py handles ALL models with CRUD operations
3. **Service Manager**: Centralized initialization order and dependencies
4. **Flat modules**: domain/users/service.py instead of nested entities/aggregates
5. **Models in application/**: Pydantic models serve as the application layer
6. **Infrastructure singletons**: Each service exports a singleton instance

---

## Technology Stack

### Backend

- Python 3.12+
- FastAPI (async)
- SQLAlchemy (async ORM) - database vendor agnostic, works with PostgreSQL, MySQL, SQLite, etc.
- Alembic (migrations)
- Pydantic (validation and DTOs)

### Frontend

Choice: Next.js 14+ with App Router

Reasoning:
- PWA support out of the box via next-pwa
- App Router maps well to RBAC structure with route groups
- Server Components for performance, Client Components for interactivity
- API route handlers can proxy to Python backend
- TypeScript support throughout
- shadcn/ui for accessible components
- Works on desktop and mobile through responsive design (single codebase)

### AI/MCP

- Separate MCP server process (Python)
- Communicates with main API via HTTP (never direct DB access)
- Respects same RBAC as requesting user
- Supports OpenAI, Anthropic, or local LLMs

### Infrastructure

- Docker and Docker Compose
- Redis for caching and sessions
- Nginx as reverse proxy

---

## Project Structure

```
school-info-system/
├── apps/
│   ├── api/                    # Python FastAPI backend
│   │   ├── src/
│   │   │   ├── application/    # Pydantic models and DTOs
│   │   │   ├── domain/         # Business logic services
│   │   │   │   ├── config/     # Settings, constants, security
│   │   │   │   ├── auth/       # Auth decorators
│   │   │   │   ├── users/      # User service
│   │   │   │   ├── academics/  # Grades, assignments service
│   │   │   │   ├── attendance/ # Attendance service
│   │   │   │   ├── scheduling/ # Schedule service
│   │   │   │   ├── messaging/  # Messaging service
│   │   │   │   └── shared/     # Exceptions, validators
│   │   │   ├── infrastructure/ # External concerns
│   │   │   │   ├── singleton.py
│   │   │   │   ├── database/   # Core, repository, models
│   │   │   │   ├── auth/       # OAuth, JWT
│   │   │   │   ├── rbac/       # RBAC enforcement
│   │   │   │   ├── ai/         # AI/MCP client
│   │   │   │   ├── audit/      # Audit logging
│   │   │   │   └── logging/    # Structured logging
│   │   │   └── presentation/   # API endpoints
│   │   ├── alembic/            # Migrations
│   │   └── seed/               # Seed data
│   │
│   ├── mcp-server/             # MCP AI server (separate process)
│   │   └── src/
│   │       ├── server.py
│   │       ├── tools/          # AI tool definitions
│   │       ├── llm/            # LLM client
│   │       ├── api_client.py   # HTTP client to main API
│   │       └── guardrails.py   # AI safety
│   │
│   └── web/                    # Next.js PWA frontend
│       └── src/
│           ├── app/            # App Router pages
│           ├── components/
│           ├── lib/
│           ├── hooks/
│           └── types/
│
├── docker/
├── docs/
└── README.md
```

---

## Pre-configured Seed Data

The system ships with Lithuanian educational standards pre-configured. Schools do not need to set these up.

### Grade Types

| Code | Lithuanian | English |
|------|------------|---------|  
| A | Atsiskaitymas | Assessment |
| D | Diktantas | Dictation |
| K | Kontrolinis darbas | Test (Major) |
| L | Laboratorinis darbas | Lab Work |
| PD | Praktinis darbas | Practical Work |
| PR | Projektinis darbas | Project Work |
| RA | Rašinys | Essay |
| S | Savarankiškas darbas | Independent Work |
| T | Testas | Quiz |
| TD | Teorinis darbas | Theory Work |
| EG | Egzaminas | Exam |
| PG | Pusmečio pažymys | Semester Grade |
| MG | Metinis pažymys | Annual Grade |

### Attendance Statuses

| Code | Lithuanian | English |
|------|------------|---------|
| P | Dalyvauja | Present |
| N | Nedalyvauja | Absent |
| V | Vėluoja | Late |
| PT | Pateisinta | Excused |
| L | Liga | Sick |
| VR | Varžybos/Renginys | Competition/Event |

### Standard Subjects

Languages:
- LT: Lietuvių kalba ir literatūra
- EN: Anglų kalba
- RU: Rusų kalba
- DE: Vokiečių kalba
- FR: Prancūzų kalba

STEM:
- MA: Matematika
- FI: Fizika
- CH: Chemija
- BI: Biologija
- IN: Informatika
- GE: Geografija

Social Sciences:
- IS: Istorija
- PL: Pilietiškumo pagrindai
- EK: Ekonomika
- PS: Psichologija

Arts:
- DA: Dailė
- MU: Muzika
- TE: Technologijos

Physical Education:
- KU: Kūno kultūra
- SV: Sveikatos ugdymas

Values:
- ET: Etika
- TI: Tikyba

Other:
- KL: Klasės valandėlė

### Relationship Types

- mother: Mama
- father: Tėtis
- guardian: Globėjas
- grandmother: Močiutė
- grandfather: Senelis
- other: Kitas

### What Schools Configure Themselves

- Schedule template (period times)
- School information (name, address, code)
- Classes with any identifier (5a, 5b, "Saulės klasė", etc.) - flexible naming, no predefined grade levels
- Teachers
- Students with class assignments
- Teacher-subject-class assignments
- Parent accounts linked to students
- Translation overrides (optional)

---

## Translation System

Decision: Database-only approach (no static JSON files, no fallbacks).

Since this is an always-online web application, translations are fetched from the API on app load.

### Translation Table Structure

```
translations:
- id (UUID)
- key (string, indexed)
- locale (string, indexed) - e.g., "en", "lt", "pl"
- school_id (UUID, nullable, indexed) - NULL for system defaults
- value (text)
- created_at
- updated_at

Unique constraint: (key, locale, school_id)
```

### Lookup Order

1. School-specific override for locale (school_id + locale)
2. System default for locale (school_id = NULL + locale)

No fallback to English needed since system ensures defaults exist.

### Features

- Multiple languages supported
- School-specific terminology overrides
- Admin UI for managing translations
- Non-developers can add/edit translations

---

## AI Features

### Approach

AI features will be developed as an addition to the core SIS API. The specific tools and capabilities will be determined as development progresses.

The general approach will be a unified chat agent interface where users can interact with AI naturally, and the system selects appropriate tools based on the request. The MCP server will communicate with the main API via HTTP (never direct DB access) to ensure RBAC is always enforced.

### Specific Tools

To be determined. AI tools will be designed and implemented after core SIS functionality is complete.

### AI Safety Constraints (Required)

- All AI suggestions require human confirmation
- AI respects same RBAC as requesting user
- All AI actions logged for audit
- Homework helper uses Socratic method only
- Confidence scores shown for suggestions
- Easy override/dismiss for AI suggestions

### MCP Server Structure

To be determined. Will follow a flat structure (not domain-ized) with MCP server calling main API for all data access to ensure RBAC is always enforced.

---

## Development Phases

### Phase 1: Foundation (Week 1-2)

- API skeleton and project structure
- SingletonServiceBase implementation
- Database core with generic repository
- Basic models (User, Role, Permission)
- Health endpoint
- JWT authentication
- RBAC system with @requires_permission decorator
- Seed data runner

### Phase 2: Core Features (Week 3-4)

- School structure (School, Class, Subject models)
- Student-Class, Teacher-Subject relationships
- Parent-Student linking
- Grade service
- Assignment service
- Attendance service with scoped permissions

### Phase 3: Frontend (Week 5-6)

- Next.js PWA setup
- Authentication flow
- Role-based navigation
- Dashboard per role
- Core data views (grades, attendance, schedule)

### Phase 4: Advanced Features (Week 7-8)

- Messaging system with permission-based recipients
- Scheduling with substitutions
- Audit logging
- Reports

### Phase 5: AI Integration (Week 9-10)

- MCP server setup
- Tool implementations
- Unified chat agent
- Frontend AI components

---

## API Design

API is in English for developer clarity. UI translations are handled by the translation system.

### Key Endpoints

| Prefix | Description |
|--------|-------------|
| /auth | Authentication and sessions |
| /users | User management |
| /grades | Grade operations |
| /attendance | Attendance tracking |
| /schedules | Timetables and substitutions |
| /messages | Internal messaging |
| /assignments | Homework and tasks |
| /classes | Class management |
| /translations | Translation management |
| /ai/chat | Unified AI chat interface |
| /admin | Administrative operations |
| /config | Configuration and health |

### Security

- JWT tokens with refresh rotation
- Password hashing with bcrypt
- RBAC enforced at middleware level
- Audit logging for sensitive actions
- CORS configuration
- Rate limiting
- Input validation and sanitization

---

## Frontend Decisions

### PWA for Desktop and Mobile

Single Next.js codebase serves both desktop and mobile through responsive design:

- Desktop: Full sidebar, wide tables, multi-column layouts
- Mobile: Bottom navigation, card layouts, single column

PWA features:
- Install to home screen (like native app)
- Offline-capable (cached pages)
- Push notifications
- Full screen mode

### UI Components

- Tailwind CSS for styling
- shadcn/ui for accessible, customizable components (code owned, not library)
- React Query for data fetching and caching

### AI Chat Panel

Floating chat panel accessible from any page:
- Natural language input
- Context-aware responses
- Tool execution results displayed inline
- Confirmation buttons for actions
- Quick action shortcuts pre-fill chat

---

## Summary of Key Decisions

1. **Architecture**: Service-oriented, EchoTuner-inspired flat structure
2. **Database**: SQLAlchemy async ORM (vendor agnostic), generic repository pattern
3. **Auth**: JWT with RBAC enforced at middleware level
4. **Frontend**: Next.js PWA (single codebase for desktop/mobile)
5. **AI**: Unified chat agent approach, specific tools TBD, never bypasses RBAC
6. **Translations**: Database-only, no static files, school overrides supported
7. **Seed data**: Lithuanian standards pre-configured (no grade levels - flexible class naming)
8. **MCP structure**: TBD, will call main API for all data (no direct DB)

---

## Next Steps

Ready to begin Phase 1: API Foundation
- Create project structure
- Implement SingletonServiceBase
- Set up database core and repository
- Create initial models
- Build authentication
- Implement RBAC system
