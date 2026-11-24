# Project Proposal: YOUI
**An Agentic AI System That Designs a Unique Frontend for Every Prompt**

**Course:** Management Information Systems in the Age of Agentic AI  
**Academic Year:** 2026/2027  
**Department:** Faculty of Science, Computer Science Department  
**Supervisor:** Prof. Sameh Ahmed (`sameh@sci.cu.edu.eg`)  

---

## 1. Title
**YOUI: An Agentic AI System That Designs a Unique Frontend for Every Prompt**

---

## 2. Introduction and Background
Large language models (LLMs) can now turn a plain-language description into working frontend code in seconds. This has made website creation available to people who cannot design or code, such as small business owners, students, and founders. The code that comes back, however, tends to look the same regardless of the request. A coffee roastery, a cybersecurity dashboard, and a children's book portfolio often receive the same skeleton: a hero section, three feature cards, and a footer, styled with default fonts and safe gradients.

This sameness is a systems problem, not a model problem. A single prompt-and-response call has no memory of what it produced before, no step that reads the subject and mood before choosing a style, and no step that checks whether the result is distinctive, fits the request, and works on a phone. Lecture 1 of this course describes the same limits of a standalone LLM: it cannot use tools, keep memory, or evaluate its own output. These gaps are closed by building a system around the model rather than by using a bigger model.

This project builds YOUI, an agentic AI system that supports a real business process: turning a client's brief into a first design draft. The agent reads the brief, chooses a design direction (palette, typography, layout structure, and visual style), generates a working frontend, checks the result with automated tools, refines it, and remembers earlier designs so that it does not repeat itself. A human reviewer approves each draft before it is released.

---

## 3. Problem Statement
The project addresses the following key challenges:
- **Repeated layouts:** AI-generated frontends reuse one page skeleton regardless of the topic.
- **Generic styling:** Default fonts, stock palettes, and safe gradients ignore the subject, brand, and audience.
- **The prompt barely matters:** The description changes text but rarely changes layout, colors, or typography.
- **No self-checking and no accountability:** A single LLM call cannot judge whether its own output is original, matches the request, works on mobile, or meets accessibility standards.
- **No measurement:** Claims of "unique designs" are rarely tested with measurable metrics.

---

## 4. Proposed Solution

### 4.1 Business Process: "Brief to First Design Draft"

| Step | As-is (Manual) | To-be (with YOUI) |
|:---|:---|:---|
| **1. Brief** | Client sends a brief by email/chat; account manager asks follow-ups. | Account manager enters brief and optional brand details in one guided form. |
| **2. Draft** | Designer chooses template and builds draft (hours). | Agent chooses design direction, avoids past designs via memory, generates draft (minutes). |
| **3. Review** | Internal review, rework, client review, more rework. | Agent checks mobile layout, contrast, code validity, and uniqueness, refining up to 3 times. |
| **4. Approval** | Informal approval over email. | Account manager approves, requests changes, or rejects with one click; decision logged. |
| **5. Handoff** | Designer finalizes and sends to client. | Designer polishes the approved draft and sends to client. |

*Human handoffs fall from ~5 to 2.*

### 4.2 Five-Stage Pipeline (Perceive, Cognize, Act)
1. **Understand:** Reads subject, audience, mood, constraints, and optional brand details (structured extraction).
2. **Design Direction:** Generates design tokens (palette, type pairing, spacing, layout archetype, style direction).
3. **Generate:** Writes the frontend (React / Tailwind / HTML / CSS / JS) from the design plan.
4. **Review:** Automated checks for mobile layout, contrast, console errors, brand fidelity, and similarity to earlier designs.
5. **Refine & Approve:** Fixes issues and regenerates (up to 3 iterations), then escalates to human reviewer.

### 4.3 Mechanisms That Prevent Repeated Designs
- **Design Tokens per prompt:** Generated fresh from prompt mood, never hardcoded.
- **Layout Archetypes:** Library of page structures (editorial, bento, split-screen, dashboard).
- **Style Directions:** Diverse visual treatments (minimal, brutalist, glassmorphism, retro).
- **Anti-Repetition Memory:** Approved designs stored with tokens, structural fingerprints, and vector embeddings in Chroma / MongoDB. Designs too similar are rejected and redone.

---

## 5. Objectives & Measurable Success Criteria

| Area | Target |
|:---|:---|
| **Uniqueness** | Mean pairwise similarity across designs for different prompts is at least 30% lower for full YOUI than for the plain LLM baseline on a 20-prompt test set. |
| **Prompt Fidelity** | Average rubric score of >= 4.0 / 5 (subject, mood, audience), at least 0.5 higher than baseline. |
| **Code Quality** | >= 90% of designs render without console errors and show no horizontal overflow at 375px width; >= 90% text meets WCAG AA contrast. |
| **Human Rating** | Blind comparison by >= 10 raters; full YOUI preferred over baseline in >= 60% of comparisons. |
| **Reliability & Safety** | Loop terminates within 3-iteration cap; 100% runs logged; safe-code filter blocks unauthorized scripts. |
| **Human Approval** | Every exported design has a logged human approval; review time reported. |

---

## 6. Team Members

| Name | Role / Focus |
|:-----|:-------------|
| **Mohamed Abobakr Ahmed** | Team Lead / Agent & Orchestration |
| **Mohamed Sayed Mohamed** | Discovery Workflow & LangGraph Agent 1 |
| **Abdelrahman Mohamed Hassan** | Frontend & Landing Page |
| **Fatma Abdelmotagly** | Auth Backend, Security & Sessions |
| **Dina Ebrahem** | Auth Frontend, User Management & UI |

> **Supervisor:** Prof. Sameh Ahmed — Faculty of Science, Computer Science Department
