# Software Requirements Specification (SRS)
## YOUI-SRS
**A two-agent system that turns a short guided conversation into a unique, ready-to-use React frontend.**

---

## 1. Introduction

The platform helps brand owners, including non-technical ones, get a unique frontend design for their brand. Agent 1 asks five simple questions with ready-made suggestions and produces a Spec. Agent 2 builds the pages from that Spec. The user then reviews the pages in a Canvas, edits them through a chat, and downloads the UI design.

### 1.1 Purpose
This document defines the functional and non-functional requirements of the system. It is the shared reference for design, development, and testing.

### 1.2 Scope

| In scope | Out of scope |
|:---|:---|
| Landing Page: who we are, the project, team information, the problem and the solution | Hosting or deploying the generated websites |
| Registration and sign-in | Backend for the generated websites |
| Chat screen with a conversation History sidebar | Blocklist management UI or an Admin panel |
| Phase 1: Agent 1 asks five questions with suggestions and produces a unique Spec | Payments or subscriptions |
| Phase 2: Agent 2 builds the React frontend pages (up to 3 pages) | |
| Phase 3: Canvas to view the pages and a chat to edit them | |
| Downloading the final UI | |

### 1.3 Definitions and Abbreviations

| Term | Meaning |
|:---|:---|
| **Agent 1** | Runs the five-question brand discovery dialog, then produces the Spec JSON. |
| **Agent 2** | Builds the React frontend code from the Spec and applies Canvas edits. |
| **Orchestrator** | The layer that controls the order of steps, runs the validation, and connects the agents. Agents never call each other directly. |
| **The five questions** | Brand identity and vibe; color palette and typography; target audience; call to action; the pages to design. |
| **Suggestions** | Ready-made answers Agent 1 proposes with each question. The user picks one or types their own answer. |
| **Spec JSON** | The JSON file holding everything Agent 2 needs to build the pages. It is also exported as a PDF. |
| **Fingerprint** | A compact signature of a Spec (template, layout, colors, fonts, section order) used to compare Specs. |
| **Uniqueness Score** | 100% minus the similarity between a new Spec and the closest existing Spec. |
| **Blocklist** | Files maintained by the team that list templates, layout patterns, and colors that must not be reused. |
| **Canvas** | An interactive area where the built pages are displayed. |
| **History Sidebar** | The panel on the left listing the user's previous conversations. |
| **LangGraph** | The framework used to model each agent workflow as a stateful graph. |
| **Guardrails** | Rules that stop an agent from performing disallowed requests. |

### 1.4 References
- IEEE 29148: Requirements Engineering
- Documentation for Python, React, MongoDB, LangChain, and LangGraph
- Documentation of the selected LLM provider

---

## 2. Overall Description

The system is a web application with three phases: brand questions, Spec generation and build, then review, edit, and download in a Canvas.

### 2.1 Users

| Type | Description | Permissions |
|:---|:---|:---|
| **End User** | A registered brand owner, usually non-technical | Chat with Agent 1, view pages in the Canvas, edit through chat, download code and the Spec, and see their own history |

*There is no Admin role. The Blocklist is a set of files maintained by the team.*

### 2.2 Technology Stack

| Layer | Technology |
|:---|:---|
| **Platform frontend** | React with Tailwind CSS |
| **Backend** | Python (FastAPI) |
| **Agents** | LangChain and LangGraph (Python) |
| **Database** | MongoDB, accessed through an async Python ODM |
| **Blocklist** | Files inside the project (`config/blocklist/`), versioned with code |
| **Language** | Multilingual (Arabic & English with RTL/LTR support) |

### 2.3 Constraints
- The Spec JSON must follow a fixed schema, because it is the only contract between the two agents.
- The Blocklist consists of files maintained by the team and shipped with each release. There is no UI to manage it.
- The uniqueness threshold is a configurable value, initially 50%.
- Uniqueness is checked against the saved Specs of all users, not only the current user.
- A user is limited to 3 frontend build attempts. A fourth attempt is allowed only after a waiting period.
- A request covers at most 3 pages (initial limit).
- The system depends on an external LLM provider, so cost and latency depend on it.

### 2.4 Assumptions and Dependencies
- The user is not technical, so all questions and suggestions use simple language.
- The LLM API is available and a fallback model exists.
- The database keeps Specs, fingerprints, and built UI versions for all users.

---

## 3. Functional Requirements

### 3.1 Landing Page and Registration

| ID | Requirement | Priority | Acceptance criterion |
|:---|:---|:---:|:---|
| **FR-01** | Display a Landing Page to any visitor with: who we are, what the project does, team information, and the problem and its solution. | Must | All four sections are visible without signing in. |
| **FR-02** | Provide register and sign-in actions on the Landing Page. | Must | The actions lead to the registration or sign-in page. |
| **FR-03** | Allow creating an account with email and password. | Must | A new account is created and can sign in. |
| **FR-04** | Allow signing in and signing out. | Must | A session is valid after sign-in and ends after sign-out. |
| **FR-05** | Prevent unauthenticated users from reaching the chat screen. | Must | Direct access redirects to sign-in. |

### 3.2 Chat Screen and History

| ID | Requirement | Priority | Acceptance criterion |
|:---|:---|:---:|:---|
| **FR-06** | After sign-in, open the chat screen with the conversation in the center. | Must | The chat is shown in the middle of the screen. |
| **FR-07** | Show a History Sidebar on the left listing the user's previous conversations. | Must | Newest first, only the user's own conversations. |
| **FR-08** | Allow opening a previous conversation and continuing it with its full state (answers, Spec, built pages). | Must | The conversation opens as it was left. |
| **FR-09** | Allow starting a new conversation. | Must | A new empty conversation appears in the History. |
| **FR-10** | Allow renaming or deleting a conversation. | Could | The change appears in the History. |
| **FR-11** | Show execution status (asking, preparing the Spec, building, ready). | Should | Status updates at each phase. |
| **FR-12** | Stream agent replies progressively. | Should | The first part of a reply appears before it completes. |

### 3.3 Phase 1: Brand Questions (Agent 1)

| ID | Requirement | Priority | Acceptance criterion |
|:---|:---|:---:|:---|
| **FR-13** | Agent 1 shall start the question dialog when the user says they want a UI for their brand. | Must | The first question appears after the user's message. |
| **FR-14** | Agent 1 shall ask five questions in order, one at a time: (1) brand identity and vibe, (2) color palette and typography, (3) target audience, (4) call to action, (5) the pages the user wants designed (maximum 3). | Must | All five questions are asked and each has a saved answer. |
| **FR-15** | With each question, Agent 1 shall offer 3 to 5 ready-made suggestions. The user can pick one or type their own answer. | Must | Suggestions appear as buttons and the text field is always available. |
| **FR-16** | Questions and suggestions shall use plain language without technical terms, each suggestion with a short description or example. | Must | Manual review of sample conversations. |
| **FR-17** | Suggestions shall depend on earlier answers and exclude Blocklist items. | Should | No suggestion matches a blocked item. |
| **FR-18** | If an answer is vague or off-topic, Agent 1 shall rephrase the question or ask for clarification. | Must | Agent 1 does not advance on an unclear answer. |
| **FR-19** | If the user chooses a blocked item or one that is too similar to an existing design, Agent 1 shall explain in simple terms why it is repeated and propose a close alternative. If the user still wants it, Agent 1 shall apply their choice as requested. | Must | The explanation is shown, and an insisted choice appears unchanged in the Spec. |
| **FR-20** | Allow the user to change an earlier answer before the questions end. | Should | The changed answer is saved and affects later suggestions. |
| **FR-21** | Offer a "choose for me" option on any question. | Could | Agent 1 picks an answer and shows it to the user. |
| **FR-22** | Save each answer as soon as it is received. | Must | Closing and reopening the page resumes at the last question. |
| **FR-23** | When all five questions are answered, Agent 1 shall leave the question loop and tell the user it is preparing the design. | Must | Leaving the loop is triggered only by the five answers being complete. |

### 3.4 Spec Generation (Agent 1)

| ID | Requirement | Priority | Acceptance criterion |
|:---|:---|:---:|:---|
| **FR-24** | Agent 1 shall convert the answers into a Spec JSON containing brand identity, palette, fonts, audience, call to action, pages, and the structure of each page (template, layout, section order, content). | Must | All required schema fields are filled. |
| **FR-25** | Before generating, Agent 1 shall load the fingerprints of earlier Specs of all users and read the Blocklist files. | Must | Fingerprints and rules are available in the generation context. |
| **FR-26** | The Spec shall have a Uniqueness Score at or above the threshold (50% by default) and contain no Blocklist item, except items the user insisted on under FR-19. | Must | The automated check passes before approval. |
| **FR-27** | The system shall validate the Spec in code (schema, uniqueness, Blocklist) and not rely on the LLM alone. | Must | A non-compliant Spec never reaches Agent 2. |
| **FR-28** | If validation fails, Agent 1 shall retry with the rejection reason, up to N times. | Must | Retries never exceed N. |
| **FR-29** | If retries are exhausted, show a simple message with a retry option and do not call Agent 2. | Must | No build happens without an approved Spec. |
| **FR-30** | Save the approved Spec and its fingerprint, linked to the user and the conversation. | Must | The record exists in the database. |
| **FR-31** | Provide the approved Spec to the user as a JSON file. | Must | The file downloads and is valid against the schema. |
| **FR-32** | Provide the approved Spec to the user as a readable PDF file. | Must | The PDF downloads and matches the JSON content. |

### 3.5 Phase 2: Build (Agent 2)

| ID | Requirement | Priority | Acceptance criterion |
|:---|:---|:---:|:---|
| **FR-33** | Agent 2 shall accept only an approved Spec. | Must | Agent 2 never runs on unapproved input. |
| **FR-34** | Agent 2 shall build UI design for each page the user selected, up to 3 pages. | Must | One built page per requested page. |
| **FR-35** | The generated UI shall follow every element of the Spec (colors, fonts, layout, sections, content, call to action). | Must | Automated review compares UI to Spec. |
| **FR-36** | Save the UI as a version linked to the Spec and the conversation. | Must | The version can be retrieved. |
| **FR-37** | Limit each user to 3 frontend build attempts. A fourth attempt is allowed only after a waiting period (T). A build attempt is one full Agent 2 run that generates UI from an approved Spec. | Must | The fourth attempt is refused before T and the user sees when it becomes available. |

### 3.6 Phase 3: Canvas and Editing

| ID | Requirement | Priority | Acceptance criterion |
|:---|:---|:---:|:---|
| **FR-38** | Display the built pages interactively in a Canvas inside the platform. | Must | The page renders and works inside the platform. |
| **FR-39** | Allow navigating between the built pages in the Canvas. | Must | Every built page can be opened. |
| **FR-40** | Provide an edit chat next to the Canvas where the user asks for changes in plain language. | Must | The request is received and processed. |
| **FR-41** | Agent 2 shall apply the edit to the current UI and refresh the Canvas. | Must | The requested change appears and the rest of the page is intact. |
| **FR-42** | If an edit changes a core Spec element (color, font, call to action), update the Spec and its fingerprint. | Should | The Spec matches the UI after the edit. |
| **FR-43** | Keep a version after each edit and allow reverting to an earlier version. | Should | Reverting restores the page as it was. |
| **FR-44** | Allow downloading the final UI design as a ZIP file. | Must | The file downloads cleanly. |

### 3.7 Blocklist

| ID | Requirement | Priority | Acceptance criterion |
|:---|:---|:---:|:---|
| **FR-45** | Read the Blocklist from files in the project (`config/blocklist/`) maintained by the team, covering template names, layout patterns, and colors. | Must | The files follow the format in Appendix A. |
| **FR-46** | Validate the Blocklist files at startup and refuse to start if a file is invalid. | Must | An invalid file stops startup with a clear message. |
| **FR-47** | Provide no UI or endpoint to change the Blocklist. | Must | No path exists to change it from the application. |

### 3.8 Agent Coordination

| ID | Requirement | Priority | Acceptance criterion |
|:---|:---|:---:|:---|
| **FR-48** | The Orchestrator shall enforce the order: Agent 1 dialog, Spec, validation, Agent 2, Canvas. | Must | Agent 2 never starts before the Spec is approved. |
| **FR-49** | Agents shall not call each other directly. Communication goes through the Orchestrator and the Spec only. | Must | Code review shows no direct calls. |
| **FR-50** | Log every step (input, output, retries, errors) per conversation. | Should | A full log can be retrieved for each conversation. |

---

## 4. Non-Functional Requirements

| ID | Category | Requirement | Priority |
|:---|:---|:---|:---:|
| **NFR-01** | Performance | The user sees a first reply or status within 3 seconds of sending a message. | Must |
| **NFR-02** | Performance | Spec generation after the fifth answer completes within (X) seconds for 90% of requests. | Should |
| **NFR-03** | Performance | Built pages appear in the Canvas within (Y) seconds for 90% of requests. | Should |
| **NFR-04** | Security | Passwords are stored with strong hashing (e.g., bcrypt or Argon2). | Must |
| **NFR-05** | Security | Sessions use JWT or secure cookies, and all traffic uses HTTPS. | Must |
| **NFR-06** | Security | LLM API keys are kept in environment variables and never exposed in code or the frontend. | Must |
| **NFR-07** | Security | Each user sees only their own conversations and data, even though uniqueness checks use all users' fingerprints. | Must |
| **NFR-08** | Security | Fingerprints used for comparison never reveal one user's content or data to another user. | Must |
| **NFR-09** | Security | Generated UI runs in the Canvas inside an isolated sandbox (iframe sandbox) so it cannot affect the platform. | Must |
| **NFR-10** | Security | Both agents are protected against prompt injection in user messages. | Must |
| **NFR-11** | Security | Per-user rate limiting. | Should |
| **NFR-12** | Reliability | Automatic retry on LLM failure and a fallback model if all retries fail. | Must |
| **NFR-13** | Reliability | Conversation state is stored in MongoDB, so it survives a dropped connection or a page refresh. | Must |
| **NFR-14** | Cost | A maximum number of tokens per request and a maximum number of retries (N). | Must |
| **NFR-15** | Scalability | Support (M) concurrent users, with components that scale independently. | Could |
| **NFR-16** | Monitoring | Logging and tracing of every step, with token and time measurements per agent. | Should |
| **NFR-17** | Maintainability | Each agent is its own module and LangGraph graph, prompts live in separate files, and the LLM sits behind an interface. | Must |
| **NFR-18** | Usability | The interface is multilingual and works on mobile. | Should |
| **NFR-19** | Privacy | Users can delete their account and data. | Should |

---

## 5. AI-Specific Requirements

### 5.1 Question Dialog (Agent 1)
- **AI-01 [Must]:** The dialog is a LangGraph workflow. Its state (answered questions, answers, current question) is persisted in MongoDB. Leaving the question loop requires all five answers to be complete, checked in code, not decided by the LLM alone.
- **AI-02 [Must]:** Each question is asked once. Agent 1 returns to questions after the loop only if the user asks to change an answer.
- **AI-03 [Should]:** Suggestions are varied and fit the brand type implied by earlier answers.
- **AI-04 [Should]:** Agent 1 replies with multilingual support.
- **AI-05 [Must]:** Custom answers typed by the user are converted into usable values in the Spec (e.g., a color name into a hex code).

### 5.2 Uniqueness
- **AI-06 [Must]:** Uniqueness Score = 100% minus the similarity between the new Spec fingerprint and the closest earlier fingerprint.
- **AI-07 [Must]:** A Spec is accepted when its score is 50% or higher (configurable), except for elements the user insisted on under FR-19.
- **AI-08 [Must]:** Comparison uses fingerprints of all users' Specs, not one user's.
- **AI-09 [Must]:** The fingerprint covers template, layout, primary color and palette, fonts, and section order.
- **AI-10 [Must]:** Colors are compared by perceptual color distance (`min_color_distance`), not by exact match.
- **AI-11 [Must]:** Comparison uses stored fingerprints, not full JSON files, to control cost and context size.
- **AI-12 [Should]:** The weight of each fingerprint element in the similarity calculation is set in a configuration file.

### 5.3 Spec JSON Contract
- **AI-13 [Must]:** The Spec JSON has a fixed, documented JSON Schema and a version number.
- **AI-14 [Must]:** Required fields include brand data, palette, fonts, audience, call to action, and the pages with the structure of each.
- **AI-15 [Should]:** Any value Agent 1 fills in because the user did not specify it is marked as `"assumed"` in the Spec.
- **AI-16 [Must]:** Agent 2 rejects any JSON that does not match the schema and does not guess missing fields.

### 5.4 Output Quality
- **AI-17 [Must]:** The Spec reflects all of the user's choices (identity, colors, audience, call to action, pages).
- **AI-18 [Must]:** A Canvas edit changes only what was requested and does not break the rest of the page.
- **AI-19 [Could]:** Generated UI contains no visible placeholder text (such as Lorem ipsum) unless the user gave no content.

### 5.5 Guardrails
- **AI-20 [Must]:** Agent 1 politely refuses requests that are not about designing a UI for a brand.
- **AI-21 [Must]:** The system refuses to generate pages with harmful or illegal content, or that impersonate a real organization.
- **AI-22 [Must]:** User messages are treated as data, not as instructions to the system.

### 5.6 Evaluation
- **AI-23 [Should]:** A fixed test set of conversations and brand types to measure uniqueness, code correctness, and suggestion quality.
- **AI-24 [Should]:** After every change to prompts or model, measure: Blocklist rejection rate, extra retry rate, average Uniqueness Score, and build success rate.

---

## 6. System Interfaces

### 6.1 User Interface
- **Landing Page:** Who we are, the project, team information, problem & solution, register & sign-in buttons.
- **Register / Sign in:** Email and password forms and validation messages.
- **Chat screen:** Conversation with Agent 1 in the center (questions and clickable suggestions) and History Sidebar on the left.
- **Canvas:** Built pages, an edit chat, page navigation, and download buttons for code (ZIP) and Spec (JSON and PDF).

### 6.2 API Endpoints
- `POST /auth/register`, `POST /auth/login`, `POST /auth/logout`
- `GET /conversations`, `POST /conversations`
- `GET /conversations/{id}`
- `POST /conversations/{id}/messages` (streamed replies)
- `GET /conversations/{id}/spec?format=json|pdf`
- `POST /conversations/{id}/builds` (subject to 3-attempt limit)
- `GET /conversations/{id}/builds`
- `POST /conversations/{id}/edits`
- `POST /conversations/{id}/builds/{version}/revert`
- `GET /conversations/{id}/download` (ZIP)
- `DELETE /users/me`

### 6.3 MongoDB Collections
1. `users` — User credentials and preferences.
2. `conversations` — Title, current phase, question and answer state.
3. `messages` — Chat messages history.
4. `specs` — Approved UI specifications.
5. `fingerprints` — Spec fingerprints for uniqueness scoring.
6. `builds` — Generated UI versions and attempt logs.

---

## 7. Use Cases

- **UC-01: Register and Sign In** — Create account or log in; open chat screen upon success.
- **UC-02: Brand Questions (Phase 1)** — 5 guided questions with suggestions; answers saved immediately.
- **UC-03: Generate and Validate Spec** — Code-level schema, Blocklist, and uniqueness checks (threshold >= 50%).
- **UC-04: Build Pages (Phase 2)** — Agent 2 builds UI for up to 3 pages; max 3 build attempts per user.
- **UC-05: Review and Edit Pages (Phase 3)** — Live preview in Canvas, natural language edits via chat, version history.
- **UC-06: Download Design** — Download final UI code as ZIP or Spec as PDF/JSON.
- **UC-07: Return to Previous Conversation** — Load conversation state, questions, answers, and Canvas from sidebar.

---

## 8. Appendices

### Appendix A: Blocklist Files (`config/blocklist/`)
- `templates`: Names of templates that are overused.
- `layouts`: Names of repeated layout patterns.
- `colors`: Banned palettes, primary colors, and minimum perceptual color distance.

### Appendix B: Fingerprint Schema (JSON)
```json
{
  "spec_id": "uuid",
  "template": "portfolio-grid-light",
  "layout_pattern": "sidebar-nav-masonry",
  "primary_color": "#0F766E",
  "palette": ["#0F766E", "#F59E0B", "#111827"],
  "fonts": ["Cairo", "Inter"],
  "section_order": ["hero", "gallery", "about", "contact"],
  "created_at": "2026-11-21T00:10:00Z"
}
```

### Appendix C: Spec JSON Schema Example
```json
{
  "schema_version": "1.0",
  "brand": { "name": "Brand Name", "vibe": ["warm", "friendly", "modern"] },
  "palette": {
    "primary": "#0F766E", "secondary": "#F59E0B",
    "background": "#FFFFFF", "text": "#111827"
  },
  "typography": { "heading": "Cairo", "body": "Inter" },
  "audience": "Young professionals, 25-35",
  "cta": { "label": "Start now", "action": "signup" },
  "pages": [
    {
      "name": "home",
      "template": "portfolio-grid-light",
      "layout_pattern": "sidebar-nav-masonry",
      "sections": [
        { "type": "hero", "content": { "title": "...", "subtitle": "..." } },
        { "type": "gallery", "content": {} }
      ]
    }
  ],
  "assumed_fields": ["typography.body"]
}
```
