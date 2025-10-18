# Task Manager Web App — Product Requirements Document (PRD)

> **Purpose**: This document tells the developer exactly what to build: features, architecture, APIs, UX, and acceptance criteria for a full task-management website with some exclusive/novel capabilities.

---

## 1) Product Overview

* **Name (placeholder):** FocusFlow (changeable)
* **Elevator pitch:** A fast, opinionated task manager that helps individuals and teams capture, organize, execute, and review work with automation and AI assistance.
* **Primary users:**

  1. **Individuals** (students, creators, freelancers).
  2. **Teams** (2–50 members) needing lightweight project management.
* **Platforms:** Responsive web app (desktop-first), PWA for offline and install.

## 2) Business Goals & Success Metrics

* **Goals:**

  * Time-to-first-task < 20 seconds for new users.
  * Daily Active Users / Weekly Active Users ratio ≥ 55%.
  * 80% of users complete at least one task within 48 hours of signup.
* **North-star metric:** Tasks completed per user per week.

## 3) User Roles & Permissions

* **Role types:**

  * **Owner:** Full control of workspace.
  * **Admin:** Manage users, settings, billing.
  * **Member:** Create/assign tasks, projects.
  * **Guest:** Commenter/assignee with restricted access.
* **Permission rules:** Project-level access control lists (ACL) + task visibility (private, project, workspace).

## 4) Core Feature Set

1. **Auth & Accounts**

   * Email + password, OAuth (Google, Apple, Microsoft), magic link.
   * 2FA (TOTP) and session management.
2. **Workspaces & Projects**

   * Multiple workspaces per user.
   * Projects with statuses (Active, On Hold, Done) and color labels.
3. **Tasks**

   * Fields: title, description (rich text/mentions), assignee(s), reporter, status (customizable Kanban states), priority, labels, due date, start date, estimate, time spent, checklist, attachments, recurrence.
   * Subtasks (nested up to 3 levels) and dependencies (blocks/blocked-by).
   * Quick add (⌘K panel) from anywhere.
4. **Views**

   * **List, Board (Kanban), Calendar, Timeline/Gantt, My Tasks, Inbox, Today/This Week, Backlog.**
   * Saved views with filters/sorts; shareable links.
5. **Comments & Activity**

   * Threaded comments, @mentions, reactions, system activity log per task.
6. **Notifications**

   * In-app, email, and optional push (PWA). Granular settings.
7. **Search**

   * Full-text search + advanced filters (assignee, label, status, due, project).
8. **Files**

   * Upload/download attachments; integration with Google Drive & OneDrive (v2).
9. **Time Tracking**

   * Start/stop timers per task; manual entries; exportable reports.
10. **Reporting & Dashboards**

* Workspace dashboard: burnup/burndown, velocity, on-time rate, workload by user, cycle time.

## 5) Exclusive / Differentiating Features

1. **Focus Mode (Flow Timer + Auto-Queue)**

   * One click starts a focus session. The system auto-generates a sequence of next actions based on priority, due dates, and dependencies.
   * During focus, notifications are suppressed and the UI shows only the current task + next.
2. **Smart Breakdown (AI)**

   * Given a task, auto-propose subtasks, estimates, and checklists. User approves/edits before creation.
3. **Priority Score (PS)**

   * Calculated from (urgency, impact, effort, dependencies, recency). Exposed as a number 0–100 with color coding.
4. **Habitify Tasks**

   * Convert any task into a habit with streaks, reminders, and weekly targets.
5. **Auto-Recurrence by Completion Pattern**

   * The system learns your cadence (e.g., every ~9 days) and suggests recurrence rules.
6. **Workflow Automations (No-code Rules)**

   * Triggers (task created/updated/overdue), Conditions (label X, assignee Y), Actions (move to column, set priority, notify). Prebuilt recipes library.
7. **Inbox from Anywhere**

   * Unique email address per workspace; emails to that address become tasks with parsed metadata.
8. **Review Week**

   * Guided weekly review: what shipped, what slipped, top blockers; auto-generated summary PDF.

## 6) Non-Functional Requirements

* **Performance:** P95 page load < 2.5s on 3G Fast; interactions < 100ms for common operations.
* **Scalability:** 10k users per workspace; 1M tasks per workspace target.
* **Security:** OWASP Top 10; encrypted at rest (AES-256) and in transit (TLS 1.2+); role-based access control; audit logs (Owner/Admin view).
* **Privacy:** GDPR/CCPA-ready data export & deletion; data residency note if using regioned DB in future.
* **Accessibility:** WCAG 2.1 AA (keyboard nav, ARIA landmarks, color contrast, focus indicators).
* **Localization:** i18n framework with English initially; RTL support ready.
* **Offline:** PWA caching; read + queued writes for tasks/comments/timers.

## 7) Suggested Tech Stack

* **Frontend:** Next.js 15 (App Router), React 18, TypeScript, Tailwind CSS, shadcn/ui, TanStack Query, Zustand.
* **Backend:** Go (Golang) + Fiber/Gin OR Node.js (NestJS) — prefer **Go**.
* **DB:** PostgreSQL 15 + Prisma (if Node) or GORM/SQLC (if Go). Redis for cache/queues.
* **Search:** PostgreSQL full-text (v1), upgrade to OpenSearch (v2) if needed.
* **Auth:** Clerk/Auth0 OR custom JWT + TOTP via Authenticator app.
* **File storage:** S3-compatible (e.g., Cloudflare R2) with presigned URLs.
* **Infra:** Docker; deploy on Fly.io/Render/Vercel (FE) + Railway/Supabase/Neon (DB) or managed Postgres.
* **Emails:** Postmark/SendGrid. **Background jobs:** BullMQ/Asynq.
* **Analytics:** PostHog.

## 8) Data Model (Initial)

**Users** `(id, email, name, avatar, locale, created_at)`
**Workspaces** `(id, name, owner_id, plan, created_at)`
**WorkspaceMembers** `(workspace_id, user_id, role)`
**Projects** `(id, workspace_id, name, status, color, created_at)`
**Tasks** `(id, project_id, workspace_id, title, description, status, priority, priority_score, due_at, start_at, estimate_minutes, time_spent_minutes, created_by, assignee_id, is_private, created_at, updated_at)`
**TaskAssignees** `(task_id, user_id)` (for multi-assignee)
**Subtasks** `(id, task_id, title, status, order)`
**Dependencies** `(task_id, depends_on_task_id)`
**Labels** `(id, workspace_id, name, color)`
**TaskLabels** `(task_id, label_id)`
**Comments** `(id, task_id, user_id, body, created_at)`
**Timers** `(id, task_id, user_id, started_at, stopped_at, duration)`
**Files** `(id, task_id, user_id, url, size, mime, created_at)`
**Rules** `(id, workspace_id, name, trigger, condition_json, action_json, enabled)`
**Invites** `(id, workspace_id, email, role, token, expires_at, accepted_at)`
**AuditLogs** `(id, workspace_id, actor_id, action, entity_type, entity_id, diff_json, created_at)`

## 9) REST API (v1) — Examples

> Base URL: `/api/v1`

**Auth**

* `POST /auth/signup` `{email, password, name}` → `{token, user}`
* `POST /auth/login` `{email, password}` → `{token}`
* `POST /auth/magic` `{email}` → `{ok}`
* `POST /auth/2fa/enable` `{totp_secret}` → `{qr, recovery_codes}`

**Workspaces & Projects**

* `GET /workspaces` → list
* `POST /workspaces` `{name}` → workspace
* `POST /workspaces/:id/invite` `{email, role}`
* `GET /projects?workspace_id=`
* `POST /projects` `{workspace_id, name, status, color}`

**Tasks**

* `GET /tasks?workspace_id=&project_id=&assignee_id=&q=&status=&label=&due_before=&due_after=` → paginated list
* `POST /tasks` `{...fields}` → task
* `PATCH /tasks/:id` `{...partial}` → task
* `POST /tasks/:id/subtasks` `{title}` → subtask
* `POST /tasks/:id/dependencies` `{depends_on_task_id}`
* `POST /tasks/:id/comments` `{body}`
* `POST /tasks/:id/timer/start`
* `POST /tasks/:id/timer/stop`

**Labels & Files**

* `POST /labels` `{workspace_id, name, color}`
* `POST /files/presign` `{filename, mime, size}` → `{upload_url, file_url}`

**Rules (Automations)**

* `POST /rules` `{workspace_id, name, trigger, condition_json, action_json, enabled}`

**Search**

* `GET /search?q=` → tasks, projects, users

**Reports**

* `GET /reports/workspace/:id/summary?range=last_30_days`

> **Note:** Provide OpenAPI/Swagger for all endpoints; include pagination, rate limits, and error codes.

## 10) Frontend UX Requirements

* **Design language:** Clean, high contrast, keyboard-driven. Light & dark themes.
* **Navigation:** Left sidebar (Workspaces, Projects, Views). Top bar with global search (⌘K), quick add, profile menu.
* **Task drawer:** Slide-over with tabs: Details, Subtasks, Activity, Files, Timer.
* **Board:** Draggable columns; WIP limits; swimlanes by assignee/label.
* **Calendar/Timeline:** Drag to set dates; multi-select.
* **Focus Mode:** Full-screen minimal UI with timer, next action, and distraction blocker.
* **Empty states:** Helpful copy and 1-click sample data.

## 11) Email & Notification Templates

* Verify email, magic link, invite accepted, task assigned, task due today, overdue summary, weekly review.

## 12) Integrations (Phased)

* **v1:** Google Sign-In, email-to-task, calendar iCal feed (read-only), Slack notifications (webhook), Drive/OneDrive attachments.
* **v2:** GitHub/Linear import, Zapier connector.

## 13) Monetization & Plans

* **Free:** 1 workspace, 3 projects, 3 automation rules, 1GB files, basic reports.
* **Pro:** Unlimited projects, advanced automations, AI breakdown, Focus Mode, 50GB files, time tracking reports.
* **Team:** SSO, audit logs, role mapping, priority support.
* Billing: monthly/yearly, proration, trial 14 days.

## 14) Security & Compliance

* JWT with short TTL + refresh tokens; revocation list in Redis.
* Password hashing with Argon2id.
* CSRF protection for cookie auth; same-site strict; HTTP-only.
* Input validation and rate limiting per IP/user.
* Org-scoped encryption keys for files (KMS if cloud provider supports it).

## 15) QA & Acceptance Criteria (Samples)

* **Create Task:** Given valid payload, task appears in list, board, and search within 1s.
* **Drag on Board:** Moving a card updates `status` and persists on refresh.
* **Focus Mode:** Starting a session hides non-essential UI and starts a timer; pausing resumes the same task; completing jumps to next queued task.
* **Automations:** Rule “When task labeled Urgent → set priority High and notify assignee” triggers within 5s.
* **Offline:** With network off, creating a task queues it; when online, it syncs and shows a “synced” badge.

## 16) Deliverables

* Source code (frontend & backend) with README and environment templates.
* Dockerfiles + docker-compose for local dev.
* Database migration scripts & seed data.
* OpenAPI JSON/YAML + Swagger UI at `/docs`.
* E2E tests (Playwright) for critical flows; unit tests for business logic.
* Staging environment URL + admin credentials (seeded test workspace).

## 17) Project Plan & Milestones (8–10 weeks example)

1. **Week 1–2:** Auth, workspaces, projects, basic tasks, list view.
2. **Week 3–4:** Board, task drawer, comments, files, search.
3. **Week 5:** Calendar/timeline, notifications, time tracking.
4. **Week 6:** Focus Mode, Priority Score, automations (MVP).
5. **Week 7:** Dashboards & reports, email-to-task, PWA/offline.
6. **Week 8:** Hardening, accessibility, tests, docs, staging.

## 18) Future Roadmap (Post-MVP)

* Native mobile apps; webhooks; advanced AI (duplicate detection, effort prediction); custom fields; forms to intake requests; import/export from Trello/Asana.

---

### Notes for the Developer

* Prefer **Go** for backend (fast, memory-safe) with clean architecture (handlers → services → repositories). Use CQRS if helpful for reporting.
* Keep feature flags for experimental modules (AI, automations). Make everything multi-tenant from day one.
* Include seeds for demo workspace and sample data to showcase features.
