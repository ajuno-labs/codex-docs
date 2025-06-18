# Feature Design

*Version 0.1 — June 17 2025*

---

## 1.1 Problem Statement / Vision

> **“Learning a new topic today feels like drowning in tabs. Codex turns the web into a *map* you can walk.”**

### Core Pain‑Points

| Pain                       | Evidence / anecdote                                                        |
| -------------------------- | -------------------------------------------------------------------------- |
| **Jargon overload**        | Learners bounce when an answer contains 5 unfamiliar acronyms.             |
| **Fragmented note‑taking** | Students keep scattered notes in Notion, PDF highlights, bookmark folders. |
| **No memory of progress**  | A week later, you forget which sub‑concepts you still don’t grasp.         |

### Vision Statement

Codex is a personal **knowledge‑graph companion**. Every answer it gives automatically becomes a node on your private map. Unknown terms are surfaced in‑context, and spaced‑repetition nudges bring them back just before you forget.

---

## 1.2 Personas & Jobs‑To‑Be‑Done

| Persona                                          | Core job                                          | Success “moment”                                                                        | Typical frustrations                             |
| ------------------------------------------------ | ------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------ |
| **Curious Learner**<br>Self‑taught dev, hobbyist | *“Explain X without losing me”*                   | Sees unknown term, clicks, instantly gets a digestible definition.                      | Wikipedia rabbit holes, jargon walls.            |
| **Undergrad Student**                            | *“Lay out everything I must review for the exam”* | Looks at his saved graph two days before finals and spots weak nodes (low familiarity). | Messy notebooks, forgetting what to revise.      |
| **Researcher / Dev**                             | *“Track APIs/papers I’ve skimmed”*                | Bulk‑imports a RFC PDF; Codex auto‑extracts spec sections into nodes.                   | Keeps 20 PDFs open; loses mental map.            |
| **Teacher / Team Lead**                          | *“See if my class/team groks the basics”*         | Opens shared graph and notices half the team marked OAuth as unfamiliar.                | Guessing knowledge gaps; writing ad‑hoc quizzes. |

---

## 1.3 Core Use‑Cases (MVP Scope)

1. **Ask → Answer**
   Free‑form question in chat pane; LLM replies in Markdown.
2. **Highlight Unknown Terms**
   Terms above user’s familiarity < 0.5 render as dotted underline.
3. **Expand Term (Branch)**
   Click term → becomes *focus node*; sidebar shows definition + links; canvas draws neighbours.
4. **Save Node Automatically**
   New focus node is inserted with `familiarity = 0.3`.
5. **Mark “Understood”**
   ✔️ button toggles to bump familiarity → 0.7 (or maxed).
6. **Review Dashboard**
   Landing page lists recent graphs, low‑familiarity nodes, upcoming spaced‑rep quizzes.

Out‑of‑scope MVP (later power features):

* Bulk PDF / YouTube import
* Shareable graphs and quizzes
* Stripe billing & team workspaces

---

## 1.4 User Journey Flow (Happy Path)

> **Scenario:** Sarah, a CS sophomore, asks “What is OAuth2 PKCE?”

1. **Landing / Dashboard**
   Sarah clicks **“New question”**.
2. **Ask**
   She types the question; hits *Enter*.
3. **Answer Pane**
   Codex replies with an overview distinguishing OAuth2 flows; terms “PKCE” and “Authorization Code” are underlined.
4. **Expand**
   She clicks **PKCE** ⇒ right sidebar shows definition; central graph pivots to PKCE node.
5. **Mark Understood**
   After reading, Sarah hits ✔️; familiarity = 0.7. A green halo appears around the node.
6. **Session End**
   She closes the tab. Codex auto‑stores chat & graph.
7. **Return Later**
   Next week, dashboard highlights **PKCE** with orange badge “review due”. Clicking opens spaced‑rep quiz.

---

## 1.5 Success Metrics & KPIs

| Metric                                 | Target @ 90 days | Why it matters                                  |
| -------------------------------------- | ---------------- | ----------------------------------------------- |
| **WAU (weekly active users)**          | 5 k+             | Baseline adoption.                              |
| **Nodes created / active user / week** | ≥ 10             | Indicates deep engagement beyond single answer. |
| **% nodes marked ✔️ within 48 h**      | ≥ 35 %           | Learners follow through (not skim‑and‑forget).  |
| **Quiz email open‑rate**               | ≥ 50 %           | Quality of spaced‑rep reminders.                |
| **Day‑30 retention**                   | ≥ 25 %           | Stickiness of personal knowledge graph.         |

*Leading indicators* such as **time‑to‑first‑branch** and **average familiarity delta per session** will be tracked via Mixpanel to spot UX frictions early.

---

### Next : Detailed flows & wireframes

> Once this feature spec is accepted, we will draft low‑fi wireframes for each screen and enumerate API contracts (*/ask*, */nodes/\:id*, */familiarity*) before diving into code.
