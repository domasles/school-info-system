# School Information System (SIS)

A modern, AI-powered school information system designed for schools with multi-language support. Built with Python FastAPI backend and a PWA frontend.

## Overview

SIS provides comprehensive school management capabilities including:

- Student and teacher management
- Grade tracking with weighted grade types
- Attendance marking and excuse workflows
- Class scheduling with substitution support
- Internal messaging system
- AI-assisted features via MCP server integration
- Role-based access control (RBAC)

## Architecture

The system follows a clean, service-oriented architecture inspired by practical patterns that minimize code duplication while maintaining clear separation of concerns.

### System Components

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                            │
│  Next.js PWA (Desktop + Mobile via responsive design)           │
└─────────────────────────────────────────────────────────────────┘
                              │ HTTPS
┌─────────────────────────────────────────────────────────────────┐
│                         API GATEWAY                             │
│  FastAPI with JWT Auth, RBAC Middleware, Audit Logging          │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                      BACKEND SERVICES                           │
│  Main API (Python/FastAPI)  │  MCP Server (AI Tools)            │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                        DATA LAYER                               │
│  PostgreSQL (via SQLAlchemy ORM)  │  Redis (Cache/Sessions)     │
└─────────────────────────────────────────────────────────────────┘
```

### Project Structure

```
school-info-system/
├── apps/
│   ├── api/                    # Python FastAPI backend
│   │   ├── src/
│   │   │   ├── application/    # Pydantic models and DTOs
│   │   │   ├── domain/         # Business logic services
│   │   │   ├── infrastructure/ # Database, auth, AI, external services
│   │   │   └── presentation/   # API endpoints
│   │   ├── alembic/            # Database migrations
│   │   └── seed/               # Seed data
│   │
│   ├── mcp-server/             # MCP AI server (separate process)
│   │   └── src/
│   │       ├── tools/          # AI tool definitions
│   │       └── llm/            # LLM client integration
│   │
│   └── web/                    # Next.js PWA frontend
│       └── src/
│           ├── app/            # App router pages
│           ├── components/     # UI components
│           └── lib/            # Utilities and API client
│
├── docker/                     # Docker compose files
└── docs/                       # Documentation
```

## Role-Based Access Control

The system implements a comprehensive RBAC model tailored to broad school structures.

### Roles

| Role | Lithuanian | Description |
|------|------------|-------------|
| Administrator | Administratorius | System administration, user management, audit logs |
| Director | Direktorius | Full visibility, minimal editing, school-wide messaging |
| Deputy | Pavaduotojas | Schedule management, substitutions, lesson distribution |
| Specialist | Specialistas | Read-only access to grades and attendance (psychologists, social pedagogues) |
| Teacher | Mokytojas | Grade entry, assignments, attendance for own subjects |
| Homeroom Teacher | Klasės auklėtojas | Teacher permissions plus class management, excuse absences |
| Parent | Tėvas/Globėjas | Read-only access to own child's data |
| Student | Mokinys | Read-only access to own data |

### Permission Scoping

Permissions are scoped to specific resources:

- **School-wide**: Director, Deputy (visibility)
- **Class-scoped**: Homeroom teacher (own class only)
- **Subject-scoped**: Teacher (own subjects only)
- **Child-scoped**: Parent (own children only)
- **Self-scoped**: Student (own data only)

## Pre-configured Data

The system ships with educational standards pre-configured:

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

### Standard Subjects

The system includes common curriculum subjects across categories:

- Languages (Lithuanian, English, Russian, German, French)
- STEM (Mathematics, Physics, Chemistry, Biology, Computer Science, Geography)
- Social Sciences (History, Civics, Economics, Psychology)
- Arts (Art, Music, Technology)
- Physical Education and Health
- Ethics and Religion

## AI Features

The MCP server provides AI-assisted functionality:

### Available Tools

| Tool | Description | Required Permission |
|------|-------------|---------------------|
| bulk_grade_class | AI-assisted grading for class submissions | grades.ai_assist |
| suggest_feedback | Generate personalized feedback | grades.ai_assist |
| bulk_mark_attendance | Mark attendance for entire class | attendance.ai_bulk |
| summarize_homework | Create assignment summaries | ai.homework_summary |
| homework_assistant | Help students understand assignments | ai.homework_help |
| draft_message | AI-draft professional messages | messages.ai_draft |
| generate_report | Generate student performance reports | ai.reports |

### AI Safety

- All AI suggestions require human confirmation
- AI respects the same RBAC as the requesting user
- All AI actions are logged for audit
- Homework assistant uses Socratic method (no direct answers)

## Translation System

The system uses a database-only translation system supporting:

- Multiple languages (initially English and Lithuanian)
- School-specific terminology overrides
- Admin UI for managing translations
- Translations fetched from API on app load
- No static JSON files or fallbacks needed

## Technology Stack

### Backend

- Python 3.12+
- FastAPI
- SQLAlchemy (async)
- PostgreSQL
- Alembic (migrations)
- Pydantic

### Frontend

- Next.js 14+ (App Router)
- React
- TypeScript
- Tailwind CSS
- shadcn/ui

### AI/MCP

- MCP Server (Python)
- OpenAI/Anthropic/Local LLM support

### Infrastructure

- Docker and Docker Compose
- Redis (caching, sessions)
- Nginx (reverse proxy)

## School Setup

After deployment, school administrators configure:

1. School information (name, address, code)
2. Schedule template (period times)
3. Classes (5a, 5b, 6a, etc.)
4. Teacher accounts
5. Student accounts with class assignments
6. Teacher-subject-class assignments
7. Parent accounts linked to students
8. Translation overrides (optional)

Estimated setup time: 40 minutes for a typical school.

## Development Phases

### Phase 1: Foundation

- API skeleton and project structure
- Database core with generic repository
- Authentication (JWT)
- RBAC system with decorators
- Seed data runner

### Phase 2: Core Features

- School structure (classes, subjects)
- User management
- Grade service
- Assignment service
- Attendance service

### Phase 3: Frontend

- Next.js PWA setup
- Authentication flow
- Role-based navigation
- Core data views and forms

### Phase 4: Advanced Features

- Messaging system
- Scheduling with substitutions
- Audit logging
- Reports

### Phase 5: AI Integration

- MCP server setup
- Tool implementations
- Frontend AI components

## API Design

The API follows RESTful conventions with consistent patterns:

- All responses include proper HTTP status codes
- Error responses follow a standard format
- Authentication via JWT in Authorization header
- RBAC enforced on all protected endpoints
- Audit logging for sensitive operations

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
| /ai | AI feature endpoints |
| /admin | Administrative operations |
| /config | Configuration and health |

## Security

- JWT tokens with refresh rotation
- Password hashing with bcrypt
- RBAC enforced at middleware level
- Audit logging for sensitive actions
- CORS configuration
- Rate limiting
- Input validation and sanitization

## License

This project is licensed under the Apache 2.0 License - see the LICENSE file for details.
