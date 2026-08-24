# SRS — general

Module: `general`
Last updated: 2025-02-14
Design: [View Design](http://localhost:8080/design/2ad71a14-4cdd-4eaf-9013-07b54aa984b3)
Design system: `design/design-system.md`

## 1. Purpose

Module serves one end-to-end greeting page for `hello-word-10`. Guest sees one centered line of text on white screen, and text comes from backend data stored in PostgreSQL. If module does not exist, product has no proof that frontend, backend, and database work together.

## 2. Actors

| Actor | Who they are | What they may do in this module |
|---|---|---|
| Guest | Any visitor with no sign-in | Open greeting page and read stored message |
| System | App runtime and backend services | Read greeting from database and return it for page render |

## 3. Scope

**In scope** — functions specified below, by plan title:

- Render greeting page

**Out of scope**

- Auth, accounts, or permissions — not part of this project.
- Editing greeting text — no admin UI or write flow is built.
- Multiple pages, navigation, or extra content — deliberate not built; project is one-screen proof of pipeline only.

## 4. Functional requirements

### 4.1 Render greeting page

**Requirement GENERAL-001 — Show stored greeting centered**

*As a* Guest, *I want to* open one page that shows greeting text from backend data, *so that* I can confirm UI, API, and database are connected.

Behaviour:

1. Guest opens page and sees one line of greeting text.
2. Page centers text horizontally and vertically in viewport.
3. Page uses white background and black text only.
4. Frontend does not hardcode displayed greeting copy.
5. Backend reads greeting from exactly one stored row in PostgreSQL and returns it to page render.

**Acceptance criteria**

| # | Given | When | Then |
|---|---|---|---|
| AC-1 | Database has one greeting row with text `Hello Word` | Guest loads page | Page shows `Hello Word` centered horizontally and vertically |
| AC-2 | Database greeting row contains any non-empty text value | Guest loads page | Page shows stored value, not hardcoded frontend copy |
| AC-3 | Page is loaded | Guest views screen | Background is white and text is black, with no extra palette or animation |
| AC-4 | Backend has no greeting row | Guest loads page | Page shows failure state instead of blank or broken content |
| AC-5 | Backend cannot read PostgreSQL | Guest loads page | Page shows failure state instead of partial content |

**Failure, boundary and permission behaviour**

| Case | Condition | Expected behaviour |
|---|---|---|
| Invalid input | Greeting text row is empty string | Request is rejected or returns failure state; blank greeting never renders |
| Boundary | Greeting text is a single short line | Page still centers line and keeps one-screen layout |
| Not found | Greeting row is absent | Page shows explicit failure state, not empty page |
| Not permitted | Guest requests page | Access is allowed; no sign-in gate exists |
| Conflict | Two actors change greeting data at same time | Last saved greeting is what page reads after write completes |
| Upstream failure | PostgreSQL unavailable | Page shows failure state and no partial greeting |

**Data touched**

| Field | Type | Required | Rule |
|---|---|---|---|
| greeting text | text | yes | One non-empty row supplies displayed copy; value may be any plain text string for this proof page |
| greeting row presence | row | yes | Exactly one active row must exist for normal render |

## 5. Screens

| Screen | Section in the design | Functions it serves | States that must exist |
|---|---|---|---|
| Greeting screen | Centered single-line page | GENERAL-001 | default, error |

## 6. Non-functional requirements

| Area | Requirement |
|---|---|
| Performance | Page renders within 2s on typical connection after API response is available |
| Accessibility | Greeting text remains readable with 4.5:1 contrast on white background |
| Responsive | Layout stays centered at 320px wide and above with no horizontal scroll |
| Localisation | Copy is English only |
| Privacy | No personal data stored or displayed |

## 7. Dependencies and assumptions

- **Depends on:** Next.js frontend, for page render.
- **Depends on:** Go backend API, for greeting retrieval.
- **Depends on:** PostgreSQL, for stored greeting row.
- **Assumption:** One greeting row is seeded before first page load; if seed is missing, page shows failure state until data exists.

| Open question | Proposed default | Who decides |
|---|---|---|
| Should empty greeting row ever be allowed? | No; empty value is invalid and must not render | Stakeholder |

## 8. Traceability

| Plan item | Requirement ids | Test cases |
|---|---|---|
| Render greeting page | GENERAL-001 | `test-cases/render-greeting-page.md` |
