# StoryCraft AI — User Stories (Story Map)

## Personas (Персоны)

| ID | Персона | Описание |
|----|---------|----------|
| **P1** | **Founder Fiona** | Фаундер стартапа без Agile-опыта. Хочет быстро структурировать идею и получить план разработки. Нетехническая. |
| **P2** | **Product Pete** | Product Owner в компании 10-200 человек. Опытный в Agile. Хочет ускорить discovery и вовлечь команду. |
| **P3** | **Team Dev** | Разработчик/тестировщик в команде. Хочет понимать контекст фич, которые ему поручают. |
| **P4** | **Coach Carla** | Agile-коуч. Проводит workshop'ы с клиентами. Нужен инструмент для фасилитации. |
| **P5** | **Analyst Alex** | Product-аналитик / BA. Нужно быстро формализовать требования из интервью. |

---

## Story Map Structure

### Backbone (Activities — верхний уровень)

```
[A1: Authenticate] → [A2: Start Discovery] → [A3: Build Story Map] → [A4: Refine with AI] → [A5: Collaborate] → [A6: Plan Release] → [A7: Export & Integrate]
```

---

## A1: Authenticate (Аутентификация)

### Tasks (MVP — Релиз 1)

**US-A1-001** As a Founder Fiona, I want to sign up with my email and password, so that I can start using the platform quickly.
- **Acceptance Criteria:**
  - Email validation in real-time
  - Password min 8 chars, 1 uppercase, 1 number
  - Confirmation email sent
  - Auto-login after signup

**US-A1-002** As a Founder Fiona, I want to sign up with my Google account, so that I don't need to create a new password.
- **Acceptance Criteria:**
  - OAuth 2.0 Google flow
  - Profile data (name, avatar) imported from Google
  - Account linked to Google for future logins

**US-A1-003** As a Product Pete, I want to log in with email/password, so that I can access my projects.
- **Acceptance Criteria:**
  - JWT token-based auth
  - "Remember me" option (30-day session)
  - Password reset via email

**US-A1-004** As any user, I want to see a landing page that explains what StoryCraft AI does, so that I understand the value before signing up.
- **Acceptance Criteria:**
  - Hero section with product video/demo GIF
  - 4 UVPs listed with icons
  - Pricing section
  - Testimonials / case studies
  - Clear CTA to sign up

### Tasks (V2 — Релиз 2)

**US-A1-005** As a Product Pete, I want to invite team members to my workspace, so that we can collaborate on story maps.
- **Acceptance Criteria:**
  - Invite via email link
  - Role selection (viewer, editor, admin)
  - Pending invites list
  - Revoke invitation

**US-A1-006** As a workspace admin, I want to manage team member permissions, so that I control who can edit or view.
- **Acceptance Criteria:**
  - Roles: Admin, Editor, Viewer
  - Permission matrix (edit map, export, invite others)
  - Transfer ownership

**US-A1-007** As any user, I want to sign up with Microsoft/Slack account, so that I can use my work credentials.
- **Acceptance Criteria:**
  - OAuth Microsoft
  - OAuth Slack

### Tasks (V3 — Enterprise)

**US-A1-008** As an enterprise user, I want SSO integration (SAML/OIDC), so that my team can use corporate auth.
- **Acceptance Criteria:**
  - SAML 2.0 support
  - OIDC support
  - Auto-provisioning (SCIM)

---

## A2: Start Discovery (Начать Discovery)

### Tasks (MVP — Релиз 1)

**US-A2-001** As a Founder Fiona, I want to describe my product idea in free text (1-3 sentences), so that the AI understands what I'm building.
- **Acceptance Criteria:**
  - Textarea with 500 char limit
  - Placeholder with example idea
  - "Continue" button enabled when text entered
  - Auto-save draft

**US-A2-002** As a Founder Fiona, I want the AI to ask me clarifying discovery questions, so that I think through important aspects of my product.
- **Acceptance Criteria:**
  - 5-7 sequential questions
  - Questions adapt based on previous answers
  - Skip option for each question
  - Progress indicator (question X of Y)
  - Questions cover: users, problem, current solutions, why now, desired outcome

**US-A2-003** As a Founder Fiona, I want to see my answers compiled into an Opportunity Canvas, so that I have a structured view of my idea.
- **Acceptance Criteria:**
  - 9-section canvas displayed:
    1. Problems or Solutions
    2. Users and Customers
    3. Solutions Today
    4. User Value
    5. User Metrics
    6. Adoption Strategy
    7. Business Problem
    8. Business Metrics
    9. Budget
  - Each section editable
  - AI-suggested content clearly marked
  - "Looks good" / "Edit" options

**US-A2-004** As a Founder Fiona, I want the AI to generate 2-4 user personas for my product, so that I understand who I'm building for.
- **Acceptance Criteria:**
  - Each persona includes: name, photo (generated), role, goals, pains, behaviors, tech proficiency
  - Personas linked to discovery answers
  - Editable fields
  - Add/remove persona option
  - Color assigned to each persona

**US-A2-005** As a Founder Fiona, I want to see a "Now Journey Map" showing how users currently solve their problem, so that I understand the existing workflow.
- **Acceptance Criteria:**
  - Left-to-right flow of current user activities
  - Pains marked with red indicators
  - Joys marked with green indicators
  - Editable steps
  - "Hot spots" — areas with high pain density highlighted

**US-A2-006** As a Founder Fiona, I want the AI to identify my biggest risks and assumptions, so that I know what needs validation.
- **Acceptance Criteria:**
  - List of risks ranked by severity (High/Medium/Low)
  - Each risk has: description, impact, validation suggestion
  - Risks linked to specific canvas sections
  - "Mark as validated" option

**US-A2-007** As a Product Pete, I want to skip the AI dialog and manually create a story map, so that I have full control when I already know what I want.
- **Acceptance Criteria:**
  - "Create blank map" option from dashboard
  - Direct access to interactive board
  - Templates gallery (empty, e-commerce, SaaS, mobile app)

**US-A2-008** As a Product Pete, I want to upload existing research (interview notes, survey results, requirements docs), so that the AI can extract insights and build an initial map.
- **Acceptance Criteria:**
  - Support .txt, .pdf, .docx, .md file upload
  - Max 10MB per file, up to 5 files
  - AI extracts: user needs, pain points, feature ideas
  - Extracted insights displayed for review
  - "Generate map from insights" button

### Tasks (V2 — Релиз 2)

**US-A2-009** As a Product Pete, I want the AI to conduct a brief market and competitor analysis, so that I understand the competitive landscape.
- **Acceptance Criteria:**
  - AI identifies 3-5 direct competitors
  - Comparison matrix: features, pricing, positioning
  - Differentiation suggestions
  - Sources listed (crunchbase, producthunt, etc.)
  - "Research is AI-generated, verify independently" disclaimer

**US-A2-010** As an Analyst Alex, I want to paste interview transcripts and have the AI extract user stories, so that I don't need to manually analyze hours of conversations.
- **Acceptance Criteria:**
  - Paste text or upload transcript file
  - AI extracts: direct quotes, needs, pain points, feature requests
  - Grouped by theme
  - Link quotes to generated stories
  - Highlight uncertainty level

**US-A2-011** As any user, I want to save multiple Opportunity Canvases, so that I can compare different product ideas.
- **Acceptance Criteria:**
  - Canvas version history
  - Duplicate canvas
  - Compare two canvases side-by-side
  - Archive old canvases

### Tasks (V3 — Релиз 3)

**US-A2-012** As a Founder Fiona, I want to speak my answers instead of typing, so that the discovery dialog feels more natural.
- **Acceptance Criteria:**
  - Voice input with speech-to-text
  - Support EN, ES, DE, FR, RU
  - Visual feedback during recording
  - Replay recorded answer

**US-A2-013** As a Product Pete, I want to integrate with Dovetail/Notion to pull research data, so that I don't need to manually export/import.
- **Acceptance Criteria:**
  - OAuth connection to Dovetail
  - OAuth connection to Notion
  - Select which projects/pages to import
  - Sync on demand

---

## A3: Build Story Map (Построить Story Map)

### Tasks (MVP — Релиз 1)

**US-A3-001** As a Founder Fiona, I want the AI to generate a story map backbone (top-level activities), so that I see the big picture of my product.
- **Acceptance Criteria:**
  - Backbone displayed as horizontal row of activity cards
  - Left-to-right narrative flow
  - Each activity: short verb phrase (e.g., "Sign Up", "Create Project", "Invite Team")
  - Activities grouped logically by user journey
  - Draggable to reorder

**US-A3-002** As a Founder Fiona, I want the AI to break down each activity into specific user tasks, so that I see the details under each step.
- **Acceptance Criteria:**
  - Tasks displayed as cards vertically under each activity
  - Task format: verb phrase (e.g., "Enter email", "Upload avatar", "Set preferences")
  - Multiple tasks per activity
  - Draggable between activities
  - Editable text

**US-A3-003** As a Founder Fiona, I want to see persona color-coding on the story map, so that I know which stories belong to which user type.
- **Acceptance Criteria:**
  - Each persona has unique color
  - Story cards show colored dot/indicator for primary persona
  - Filter by persona (show only stories for selected persona)
  - "Unassigned" indicator for stories without persona

**US-A3-004** As any user, I want to add, edit, and delete cards on the story map, so that I can customize the AI-generated content.
- **Acceptance Criteria:**
  - Add new activity (backbone level)
  - Add new task under any activity
  - Edit card text inline
  - Delete card with confirmation
  - Drag-and-drop reordering at same level
  - Drag-and-drop moving between levels

**US-A3-005** As any user, I want to zoom in and out of the story map, so that I can see either the big picture or the details.
- **Acceptance Criteria:**
  - Zoom controls (+/- buttons)
  - Mouse wheel zoom
  - Pinch-to-zoom on touch
  - Fit-to-screen button
  - Zoom levels: Overview (backbone only) → Normal → Detailed (with acceptance criteria)

**US-A3-006** As a Product Pete, I want to create a story map from scratch (without AI), so that I have full control over the structure.
- **Acceptance Criteria:**
  - Blank canvas option
  - Pre-built templates: E-commerce, SaaS, Mobile App, Marketplace
  - Template includes placeholder backbone
  - "Apply template" fills in example tasks

**US-A3-007** As any user, I want the map to auto-save as I work, so that I don't lose progress.
- **Acceptance Criteria:**
  - Auto-save every 5 seconds
  - "Last saved" timestamp
  - Recovery from unexpected disconnect
  - Conflict resolution if multiple sessions

### Tasks (V2 — Релиз 2)

**US-A3-008** As a Product Pete, I want the AI to generate "What-About" scenarios (edge cases, alternatives, errors), so that I don't miss important scenarios.
- **Acceptance Criteria:**
  - "Generate What-About" button per activity
  - AI suggests: alternative flows, error cases, edge cases
  - Suggestions marked with "AI-suggested" badge
  - One-click add to map
  - User can edit before adding

**US-A3-009** As a Product Pete, I want to add acceptance criteria to individual stories, so that the development team knows when a story is done.
- **Acceptance Criteria:**
  - Expand card to see/edit details
  - Add bullet-point acceptance criteria
  - AI can suggest criteria based on story text
  - Checkboxes for criteria completion
  - Display criteria count on card

**US-A3-010** As any user, I want to add UI sketches or reference images to story cards, so that I can link visual references.
- **Acceptance Criteria:**
  - Upload image (jpg, png, gif) max 5MB
  - Paste image from clipboard
  - Thumbnail preview on card
  - Full-size view on click
  - Figma/Miro link embed

**US-A3-011** As a Product Pete, I want to mark stories with different statuses, so that I track progress.
- **Acceptance Criteria:**
  - Statuses: Draft, Ready for Dev, In Progress, Done, Archived
  - Color coding by status
  - Status filter
  - Bulk status update

**US-A3-012** As any user, I want to search and filter the story map, so that I can find specific stories quickly.
- **Acceptance Criteria:**
  - Full-text search across all cards
  - Filter by: persona, status, AI-generated vs manual, tag
  - Highlight matching cards
  - Clear filters button

### Tasks (V3 — Релиз 3)

**US-A3-013** As a Product Pete, I want the AI to analyze dependencies between stories, so that I know what needs to be built before what.
- **Acceptance Criteria:**
  - Visual dependency arrows between cards
  - AI-detected dependencies marked
  - Manual dependency creation
  - Dependency cycle detection
  - "Blocked by" indicator

**US-A3-014** As a Coach Carla, I want to use pre-built workshop templates, so that I can quickly set up a story mapping session with clients.
- **Acceptance Criteria:**
  - Template gallery: "New Product Discovery", "Feature Planning", "Quarterly Roadmap"
  - Each template: structure, suggested time boxes, facilitation notes
  - Timer for time-boxed activities
  - Breakout section markers

---

## A4: Refine with AI (Улучшить с AI)

### Tasks (MVP — Релиз 1)

**US-A4-001** As a Founder Fiona, I want the AI to analyze my story map and find gaps (missing steps), so that my product plan is complete.
- **Acceptance Criteria:**
  - "Find Gaps" button
  - AI checks: missing auth steps, missing error handling, missing admin functions, missing onboarding
  - Gaps displayed as suggestions with explanation
  - Severity indicator (Critical / Warning / Suggestion)
  - One-click add to map

**US-A4-002** As a Founder Fiona, I want the AI to suggest improvements to my story descriptions, so that they follow best practices.
- **Acceptance Criteria:**
  - "Improve Description" per card
  - AI suggests: clearer verb phrase, adds "so that" clause
  - Before/after comparison
  - Apply or dismiss suggestion

### Tasks (V2 — Релиз 2)

**US-A4-003** As a Product Pete, I want the AI to generate acceptance criteria for any story, so that I save time on specification.
- **Acceptance Criteria:**
  - "Generate Criteria" button per card
  - 3-7 bullet-point criteria generated
  - Criteria follow Given-When-Then format option
  - Editable after generation
  - Linked to story context

**US-A4-004** As a Product Pete, I want the AI to estimate story size (S/M/L), so that I get a rough sense of effort.
- **Acceptance Criteria:**
  - T-shirt sizing: S (1-3 days), M (3-5 days), L (1-2 weeks)
  - AI estimation based on story complexity, dependencies, similar stories
  - Estimation visible on card
  - Team can override AI estimation
  - Sum of estimates per release slice

**US-A4-005** As any user, I want the AI to restructure my story map if it detects organizational issues, so that the map follows best practices.
- **Acceptance Criteria:**
  - Detect: orphan tasks (no activity), overloaded activities (>10 tasks), empty activities
  - Suggest: split activity, merge activities, reorder
  - "Apply restructure" preview before applying

### Tasks (V3 — Релиз 3)

**US-A4-006** As a Product Pete, I want the AI to suggest metrics/KPIs for each story, so that I can measure success.
- **Acceptance Criteria:**
  - "Suggest Metrics" per activity or story
  - Metrics: adoption rate, task completion, error rate, NPS
  - Baseline suggestion (current state → target state)

---

## A5: Collaborate (Коллаборация)

### Tasks (V2 — Релиз 2)

**US-A5-001** As a Product Pete, I want to invite team members to view and edit my story map in real-time, so that we can workshop together.
- **Acceptance Criteria:**
  - Share via link (view or edit)
  - Invite by email
  - Real-time cursor positions visible
  - Real-time card updates (no refresh needed)
  - "X users viewing" indicator
  - Conflict resolution (last edit wins + notification)

**US-A5-002** As a team member, I want to leave comments on story cards, so that I can discuss specifics without changing the card.
- **Acceptance Criteria:**
  - Comment thread per card
  - @mentions with notifications
  - Resolve/unresolve comments
  - Comment badge with count on card
  - Filter by resolved/unresolved

**US-A5-003** As a Product Pete, I want to run a prioritization voting session with my team, so that we can democratically decide what to build first.
- **Acceptance Criteria:**
  - "Start Voting Session" button
  - Voting methods: dot voting, MoSCoW, impact/effort matrix
  - Anonymous or named voting
  - Results visualization (bar chart, matrix)
  - Apply results to story ordering

**US-A5-004** As any user, I want to see version history of the story map, so that I can track changes and revert if needed.
- **Acceptance Criteria:**
  - Timeline of all changes
  - Who changed what and when
  - Compare two versions
  - Revert to any previous version
  - Named versions ("Before Q2 Planning")

### Tasks (V3 — Релиз 3)

**US-A5-005** As a Coach Carla, I want to present/facilitate the story map in presentation mode, so that I can walk stakeholders through it.
- **Acceptance Criteria:**
  - Presentation mode: fullscreen, clean UI
  - Navigate left-to-right with arrow keys
  - Zoom into details during presentation
  - Laser pointer / highlight tool
  - Export presentation as slides

**US-A5-006** As a Product Pete, I want to assign stories to team members, so that everyone knows their responsibilities.
- **Acceptance Criteria:**
  - Assign user to card
  - Assignment visible on card (avatar)
  - Filter "My Stories"
  - Notification on assignment
  - Due date option

---

## A6: Plan Release (Планировать Релиз)

### Tasks (MVP — Релиз 1)

**US-A6-001** As a Founder Fiona, I want the AI to suggest an MVP slice (minimum viable product), so that I know the smallest thing to build first.
- **Acceptance Criteria:**
  - Horizontal "slice line" across the map
  - AI selects stories above the line for MVP
  - Rationale provided: "These 12 stories allow a user to complete the core journey end-to-end"
  - User can drag line to adjust
  - Stories in MVP highlighted
  - MVP stats: story count, estimated duration

**US-A6-002** As a Founder Fiona, I want to see a release roadmap with multiple slices (MVP, V2, V3), so that I have a phased plan.
- **Acceptance Criteria:**
  - Multiple horizontal slice lines
  - Named releases: "MVP", "V2 — Enhancements", "V3 — Scale"
  - Release descriptions editable
  - Story count per release
  - Timeline estimate per release

**US-A6-003** As any user, I want to manually move stories between releases by dragging them, so that I have full control over the plan.
- **Acceptance Criteria:**
  - Drag story card across slice lines
  - Visual feedback during drag
  - Release totals update automatically
  - "Unassigned" area below all releases

### Tasks (V2 — Релиз 2)

**US-A6-004** As a Product Pete, I want the AI to estimate total effort for each release slice, so that I can plan capacity.
- **Acceptance Criteria:**
  - Sum of T-shirt sizes per release
  - Converted to rough sprint count (assuming 2-week sprints)
  - Team size input affects timeline
  - Consideration of parallel work

**US-A6-005** As a Product Pete, I want to define dependencies between releases, so that I know what must be built before what.
- **Acceptance Criteria:**
  - Mark release as dependent on another
  - Visual arrow between releases
  - Sequence validation (circular dependency warning)

**US-A6-006** As a Product Pete, I want to set target dates for each release, so that I have a time-bound roadmap.
- **Acceptance Criteria:**
  - Date picker per release
  - Calendar view of releases
  - Warning if estimated effort > available time
  - Milestone markers

### Tasks (V3 — Релиз 3)

**US-A6-007** As a Product Pete, I want the AI to predict outcomes for each release (user adoption, business impact), so that I can set expectations.
- **Acceptance Criteria:**
  - "Predict Outcomes" per release
  - AI estimates: user activation rate, feature adoption, time-to-value
  - Based on similar products/industry benchmarks
  - Confidence level indicator
  - Editable assumptions

---

## A7: Export & Integrate (Экспорт и Интеграция)

### Tasks (MVP — Релиз 1)

**US-A7-001** As a Founder Fiona, I want to export my story map as a PNG/PDF image, so that I can share it in presentations.
- **Acceptance Criteria:**
  - Export current view as PNG
  - Export full map as PDF (multi-page if needed)
  - Branding: StoryCraft AI watermark (removable on Pro)
  - Resolution options

**US-A7-002** As a Product Pete, I want to export stories to JIRA as Epics and User Stories, so that my dev team can start working.
- **Acceptance Criteria:**
  - OAuth connection to JIRA
  - Select project and board
  - Activities → Epics
  - Tasks → User Stories (with acceptance criteria as sub-tasks)
  - Personas → Labels
  - Release slices → Fix Versions
  - Export progress indicator
  - Link back to StoryCraft AI

**US-A7-003** As a Product Pete, I want to export stories to Linear, so that my team can work in their preferred tool.
- **Acceptance Criteria:**
  - OAuth to Linear
  - Activities → Projects
  - Tasks → Issues
  - Releases → Cycles
  - Labels preserved

**US-A7-004** As any user, I want to generate a Product Requirements Document (PRD) from my story map, so that I have a formal document for stakeholders.
- **Acceptance Criteria:**
  - PRD includes: overview, personas, story map summary, detailed stories with acceptance criteria, release plan
  - Formatted markdown document
  - Download as .md or .docx
  - Customizable template

### Tasks (V2 — Релиз 2)

**US-A7-005** As a Product Pete, I want to export to GitHub Issues, so that I can manage stories alongside code.
- **Acceptance Criteria:**
  - OAuth to GitHub
  - Select repository
  - Activities → Milestones
  - Tasks → Issues with labels
  - Acceptance criteria → Issue checklist

**US-A7-006** As a Product Pete, I want a two-way sync with JIRA, so that when stories are updated in JIRA, they reflect in StoryCraft AI.
- **Acceptance Criteria:**
  - Status sync: JIRA status → StoryCraft status
  - Comments sync option
  - Manual or automatic sync (configurable interval)
  - Conflict resolution UI

**US-A7-007** As a Product Pete, I want to export a CSV of all stories, so that I can analyze in spreadsheets or import elsewhere.
- **Acceptance Criteria:**
  - CSV columns: ID, Title, Activity, Description, Persona, Status, Release, Acceptance Criteria, Size
  - UTF-8 encoding
  - Filter before export

**US-A7-008** As any user, I want to share a public read-only link to my story map, so that I can share with external stakeholders.
- **Acceptance Criteria:**
  - Generate public link
  - Password protection option
  - Expiry date option
  - View-only (no editing)
  - Revoke link

### Tasks (V3 — Релиз 3)

**US-A7-009** As a Product Pete, I want to receive Slack notifications when my story map is exported or when team members make significant changes, so that I stay informed.
- **Acceptance Criteria:**
  - Connect Slack workspace
  - Select channel for notifications
  - Notification types: export complete, major edit, comment mention
  - Unlink option

**US-A7-010** As a Product Pete, I want to import from an existing JIRA backlog and convert it to a story map, so that I can reorganize my flat backlog.
- **Acceptance Criteria:**
  - Import epics and stories from JIRA
  - AI suggests organization into narrative flow
  - Manual adjustment after import
  - Preserve JIRA IDs for reference

---

## Release Slicing Summary

### MVP (Релиз 1) — 
## Release Slicing Summary

### MVP (Релиз 1) — "Core Discovery & Mapping" — 2-3 months

**Цель:** Пользователь может пройти discovery-диалог, получить AI-generated story map, редактировать её на доске, и экспортировать в базовые форматы.

**Stories (55):**

| ID | Story | Приоритет |
|----|-------|:---------:|
| US-A1-001 | Sign up with email/password | Must |
| US-A1-002 | Sign up with Google OAuth | Must |
| US-A1-003 | Log in with email/password | Must |
| US-A1-004 | Landing page with value prop | Must |
| US-A2-001 | Describe product idea in free text | Must |
| US-A2-002 | AI discovery dialog with adaptive questions | Must |
| US-A2-003 | Generate Opportunity Canvas (9 sections) | Must |
| US-A2-004 | AI generate 2-4 user personas | Must |
| US-A2-005 | Generate "Now Journey Map" | Must |
| US-A2-006 | Risk & assumption detection | Must |
| US-A2-007 | Skip AI dialog, create manual map | Must |
| US-A2-008 | Upload research documents for AI analysis | Must |
| US-A3-001 | AI generate story map backbone (activities) | Must |
| US-A3-002 | AI decompose activities into tasks | Must |
| US-A3-003 | Persona color-coding on map | Must |
| US-A3-004 | Add/edit/delete cards on map | Must |
| US-A3-005 | Zoom in/out of story map | Must |
| US-A3-006 | Create map from scratch (templates) | Must |
| US-A3-007 | Auto-save map | Must |
| US-A4-001 | AI gap analysis (find missing steps) | Must |
| US-A4-002 | AI suggest story description improvements | Must |
| US-A6-001 | AI suggest MVP slice | Must |
| US-A6-002 | Create release roadmap with slices | Must |
| US-A6-003 | Manually move stories between releases | Must |
| US-A7-001 | Export as PNG/PDF | Must |
| US-A7-002 | Export to JIRA (Epics + Stories) | Must |
| US-A7-003 | Export to Linear | Must |
| US-A7-004 | Generate PRD document | Must |

---

### V2 (Релиз 2) — "Team Collaboration & AI Enhancement" — 1-2 months

**Цель:** Команды могут совместно работать над картой, AI помогает с деталями, есть экспорт в популярные dev-tools.

**Stories (20):**

| ID | Story |
|----|-------|
| US-A1-005 | Invite team members to workspace |
| US-A1-006 | Manage team permissions |
| US-A1-007 | Sign up with Microsoft/Slack |
| US-A2-009 | Market & competitor analysis |
| US-A2-010 | Paste interview transcripts → extract stories |
| US-A2-011 | Save & compare multiple Opportunity Canvases |
| US-A3-008 | AI What-About scenario generator |
| US-A3-009 | Acceptance criteria per story |
| US-A3-010 | Attach images/UI sketches to cards |
| US-A3-011 | Story status tracking |
| US-A3-012 | Search & filter story map |
| US-A4-003 | AI generate acceptance criteria (Given-When-Then) |
| US-A4-004 | AI effort estimation (T-shirt sizing) |
| US-A4-005 | AI detect & suggest map restructuring |
| US-A5-001 | Real-time collaborative editing |
| US-A5-002 | Comment threads on cards |
| US-A5-003 | Team voting/prioritization session |
| US-A5-004 | Version history & revert |
| US-A6-004 | AI estimate effort per release |
| US-A7-005 | Export to GitHub Issues |

---

### V3 (Релиз 3) — "Advanced AI & Scale" — 1-2 months

**Цель:** Голосовой ввод, продвинутый AI, enterprise-готовность.

**Stories (14):**

| ID | Story |
|----|-------|
| US-A1-008 | Enterprise SSO (SAML/OIDC) |
| US-A2-012 | Voice input for discovery dialog |
| US-A2-013 | Dovetail/Notion integration |
| US-A3-013 | AI dependency mapping between stories |
| US-A3-014 | Workshop templates for coaches |
| US-A4-006 | AI suggest metrics/KPIs per story |
| US-A5-005 | Presentation mode for facilitation |
| US-A5-006 | Assign stories to team members |
| US-A6-005 | Define dependencies between releases |
| US-A6-006 | Set target dates for releases |
| US-A6-007 | AI predict outcomes per release |
| US-A7-006 | Two-way sync with JIRA |
| US-A7-007 | Export to CSV |
| US-A7-008 | Public read-only sharing link |
| US-A7-009 | Slack notifications |
| US-A7-010 | Import from JIRA backlog → story map |

---

## Story Count Summary

| Release | Stories | Focus |
|---------|:-------:|-------|
| **MVP (R1)** | 28 | Discovery dialog, AI generation, basic board, export |
| **V2 (R2)** | 20 | Collaboration, AI enhancement, more exports |
| **V3 (R3)** | 16 | Advanced AI, voice, enterprise, presentation |
| **Total** | **64** | |

---

## Appendix A: Non-Functional Requirements

### Performance
- **NFR-001:** Page load < 2 seconds for landing page
- **NFR-002:** Story map interaction (drag, zoom) < 100ms response
- **NFR-003:** AI dialog response < 3 seconds per question
- **NFR-004:** Support maps with up to 500 cards without degradation
- **NFR-005:** Real-time collaboration latency < 200ms

### Security
- **NFR-006:** All data encrypted at rest (AES-256) and in transit (TLS 1.3)
- **NFR-007:** OAuth tokens stored encrypted, refreshed automatically
- **NFR-008:** Rate limiting: 100 AI requests/hour on Free, 500 on Pro, unlimited on Team
- **NFR-009:** GDPR compliant — data export and deletion on request

### Reliability
- **NFR-010:** 99.9% uptime SLA
- **NFR-011:** Auto-save recovery after disconnect
- **NFR-012:** Daily backups, 30-day retention

### Accessibility
- **NFR-013:** WCAG 2.1 AA compliance
- **NFR-014:** Keyboard navigation for all features
- **NFR-015:** Screen reader compatible

### Usability
- **NFR-016:** Onboarding to first AI-generated map < 10 minutes
- **NFR-017:** No training required for basic usage
- **NFR-018:** Mobile-responsive (view-only on mobile, edit on tablet+)

---

## Appendix B: AI Behavior Specifications

### Discovery Dialog Engine
```
Trigger: User submits product idea description
Input: Free text (1-500 chars), persona type (founder/pm/analyst)
Process:
  1. Parse idea → extract: domain, problem statement, implied users
  2. Generate question sequence (5-7 questions)
  3. Adapt subsequent questions based on answers
  4. After all answers → generate Opportunity Canvas + Personas + Journey Map
Output: Structured discovery artifact
Constraints:
  - Questions must be in user's language
  - Max 7 questions to avoid fatigue
  - Each question has "skip" option
  - Context window: maintain full conversation history
```

### Gap Analysis Engine
```
Trigger: User clicks "Find Gaps" or auto-triggered on map change
Input: Current story map state (activities, tasks, personas)
Process:
  1. Check for: auth flow gaps, error handling, admin functions, onboarding
  2. Compare against domain-specific patterns (e-commerce, SaaS, etc.)
  3. Score severity based on criticality
Output: List of gap suggestions with severity and rationale
Constraints:
  - Max 20 suggestions per analysis
  - Explain why each gap matters
  - Link to relevant persona if applicable
```

### MVP Slicer
```
Trigger: User requests MVP suggestion
Input: Complete story map, personas, Opportunity Canvas
Process:
  1. Identify backbone activities essential for core journey
  2. Select minimum tasks per activity for end-to-end flow
  3. Exclude: nice-to-have, admin, advanced features
  4. Ensure at least one complete user journey is possible
Output: Suggested slice line with rationale
Constraints:
  - MVP must enable at least one complete user task end-to-end
  - Provide reasoning for each included/excluded story
  - Suggest 2-3 alternative MVP compositions
```

---
*Generated based on "User Story Mapping" by Jeff Patton methodology*
