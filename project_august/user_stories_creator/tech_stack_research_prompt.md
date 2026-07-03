# Tech Stack Research Prompt: StoryCraft AI

## Product Overview

StoryCraft AI is an AI-powered user story mapping and product discovery platform. It guides users (startup founders, product owners, agile coaches, analysts) through a structured product discovery process and helps them build, refine, and export interactive story maps.

## Core Functionalities Requiring Stack Decisions

### 1. AI-Powered Discovery Dialog
- Adaptive multi-turn conversation (5-7 questions) based on user's free-text product idea
- Generates structured artifacts: Opportunity Canvas (9 sections), User Personas (2-4 with photos), "Now Journey Map", Risk & Assumption list
- Must maintain full conversation context
- Voice input support (speech-to-text) in EN, ES, DE, FR, RU
- Interview transcript analysis (extract quotes, needs, pain points, feature requests)

### 2. Interactive Story Mapping Board
- Infinite canvas with horizontal backbone (activities) and vertical tasks
- Drag-and-drop cards between and within levels
- Zoom controls (overview → normal → detailed with acceptance criteria)
- Persona color-coding and filtering
- Auto-save every 5 seconds
- Support for 500+ cards without performance degradation
- Inline editing, status tracking, image attachments (up to 5MB)

### 3. Real-Time Collaboration
- Multi-user editing with live cursor positions
- Real-time card updates (no refresh)
- Comment threads with @mentions
- Voting sessions (dot voting, MoSCoW, impact/effort matrix)
- Version history with diff/compare and revert
- Conflict resolution (last edit wins + notification)

### 4. AI Refinement Features
- Gap analysis (missing auth, error handling, onboarding, admin)
- Story description improvement suggestions
- Acceptance criteria generation (Given-When-Then)
- Effort estimation (T-shirt sizing S/M/L)
- Dependency mapping between stories
- Metrics/KPI suggestions per story
- "What-About" scenario generation (edge cases, alternatives, errors)

### 5. Release Planning
- Horizontal slice lines for MVP/V2/V3
- Drag stories across release boundaries
- Effort summation per release, sprint count estimation
- Target date setting with calendar view
- Outcome prediction (adoption rate, time-to-value)

### 6. Export & Integrations
- PNG/PDF export of boards
- JIRA export (Activities→Epics, Tasks→Stories, Releases→Fix Versions) + two-way sync
- Linear export (Activities→Projects, Tasks→Issues, Releases→Cycles)
- GitHub Issues export (Activities→Milestones, Tasks→Issues)
- PRD generation (markdown/docx)
- CSV export
- Public read-only sharing links

### 7. Authentication & Teams
- Email/password, Google/Microsoft/Slack OAuth
- Enterprise SSO (SAML 2.0, OIDC, SCIM)
- Workspace/team invites with roles (Admin, Editor, Viewer)
- Permission matrix

### 8. Non-Functional Requirements
- Page load < 2s, interactions < 100ms, AI response < 3s
- 99.9% uptime SLA
- Real-time collaboration latency < 200ms
- WCAG 2.1 AA accessibility compliance
- Mobile-responsive (view-only on mobile, edit on tablet+)
- GDPR compliant
- Support for large maps (500+ cards)

---

## Research Questions

Please research and recommend the **optimal tech stack** for implementing StoryCraft AI. Address each layer below, considering the specific requirements above (especially real-time collaboration, AI integration, and interactive canvas performance).

### 1. Frontend Framework & Board Engine
- **Options to evaluate**: React, Vue, Svelte, Solid — with specific canvas libraries
- **Key requirement**: What library handles an infinite draggable/zoomable canvas with 500+ interactive nodes best? (Evaluate: React Flow, Excalidraw engine, tldraw, Fabric.js, PixiJS, raw Canvas API, Felt, BoardKit)
- How to achieve <100ms drag/zoom/resize performance at scale?
- How to sync board state in real-time across users?
- Accessibility: keyboard navigation and screen reader support for canvas-based UIs

### 2. Backend Framework
- **Options to evaluate**: Node.js/Express/NestJS, Python/FastAPI, Go, Rust, Ruby on Rails
- Which handles concurrent WebSocket connections + AI orchestration + file processing best?
- How to structure the API for CRDT/sync operations?

### 3. Real-Time Collaboration & State Sync
- **Options to evaluate**: Yjs, Automerge, Socket.io, Liveblocks, PartyKit, Ably, Pusher, custom CRDTs
- How to handle conflict resolution for card moves, edits, deletions?
- How to persist operation history for version control?
- What's the latency budget and how to stay under 200ms globally?

### 4. Database
- **Options to evaluate**: PostgreSQL, MongoDB, DynamoDB, Redis, combination
- How to model hierarchical story maps (activities → tasks → criteria) with ordering?
- How to store version history efficiently?
- How to store real-time operation logs?

### 5. AI / LLM Integration
- **Options to evaluate**: OpenAI GPT-4o/Claude 3.5 Sonnet/Gemini via API, self-hosted models, LangChain/LlamaIndex, Vercel AI SDK
- How to structure the discovery dialog as a state machine with context window management?
- How to generate structured outputs (canvas sections, personas, journey maps) reliably? (Evaluate: function calling, structured outputs, prompt chaining)
- How to handle voice input (speech-to-text)? Whisper API vs. browser-native vs. cloud provider STT
- How to process document uploads (PDF, DOCX) for insight extraction?
- Cost estimation for AI features (requests per user session)

### 6. File Storage & Processing
- Where to store user uploads (transcripts, images, documents)?
- How to process PDF/DOCX/Markdown for AI extraction?

### 7. Authentication & Authorization
- **Options to evaluate**: Auth0, Clerk, Firebase Auth, Keycloak, NextAuth, custom
- Which supports all required OAuth providers + SAML/OIDC + SCIM out of the box?
- How to model workspace/team permissions efficiently?

### 8. Export & Integration Architecture
- How to generate PDF/PNG from the canvas server-side? (Puppeteer, Playwright, html2canvas, dedicated services)
- How to implement JIRA/Linear/GitHub two-way sync reliably? (Webhooks, polling, queue-based)
- How to generate DOCX/MD documents?

### 9. Infrastructure & DevOps
- **Options to evaluate**: Vercel, Railway, Render, AWS, GCP, Fly.io
- How to deploy the real-time collaboration server globally for low latency?
- How to scale AI request processing independently from app traffic?
- Cost estimation for MVP (1000 active users) and scale (10,000 active users)

### 10. Recommended "Dream Stack"
Provide a concrete, opinionated recommendation:
- Frontend: [framework] + [canvas library] + [state management]
- Backend: [framework] + [language]
- Real-time: [library/service]
- Database: [primary] + [cache] + [queue]
- AI: [LLM provider] + [orchestration] + [voice STT]
- Auth: [service]
- File Storage: [service]
- Export: [approach]
- Hosting: [platform]

For each choice, explain **why** it's the best fit specifically for StoryCraft AI's requirements (not just general popularity).

---

## Deliverables

1. **Comparison matrix** for each layer (pros/cons relevant to our use case)
2. **Final recommended stack** with justification
3. **Architecture diagram** description (how the pieces fit together)
4. **MVP cost estimate** (monthly infrastructure + AI costs for 1000 active users)
5. **Known risks** with chosen stack and mitigation strategies
6. **Migration path**: If we start with X, how hard is it to switch to Y later?

---

## Research Methodology Notes

- Prioritize **proven solutions** for real-time collaborative canvases (this is the hardest problem)
- Prioritize **developer velocity** for MVP but ensure the foundation can scale
- Consider **ecosystem maturity** (libraries, community, hiring)
- Include **specific library names**, not just framework categories
- If evaluating self-hosted vs. managed services, include operational complexity comparison
