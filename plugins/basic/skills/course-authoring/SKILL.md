---
name: course-authoring
description: Create courses for the Course App platform using the composable DSL of atoms, molecules, and organisms
---

# Course Authoring Guide

Create courses for the Course App platform by writing **two** DSL scripts — a **question bank** and a **course** — and POSTing each to the API in order.

## Architecture Overview

Persistence is two-layered:

- **Question bank** — named collection of questions plus optional **pools** (selectors over those questions).
- **Course** — metadata (title, description, mode, passing score) grouping **topics → sections**. Sections' content trees contain `questionSlot(...)` placeholders that reference a pool by ID.
- **Slot resolution** — at first load the server picks N questions from the named pool, persists the pick against the learner's registration, and streams a resolved course to the renderer.

Questions are **never** embedded in section content. Authoring order is: POST bank → POST course (referencing pool IDs from the bank).

## Data Model

```
question_banks (1) ── questions (N)
                   └── pools (N, selectors)  ◄──┐
                                                │ referenced by pool id
courses (1) ── topics (N) ── sections (N)       │
                              │                 │
                              └── content.kids[].questionSlot(slotId, rule)
                                                │
                                  slot_resolutions (per-registration pick)
```

## Rich Text (Markdown in Strings)

All text strings in the DSL support markdown formatting. The renderer parses markdown when present — plain strings pass through unchanged.

### Supported Syntax

| Syntax | Renders As |
|--------|-----------|
| `**bold**` | bold |
| `*italic*` | italic |
| `` `code` `` | code |
| `[text](url)` | clickable link |
| `- item` or `* item` | unordered list |
| `1. item` | ordered list |
| blank line | paragraph break |

Multi-line example:

```ts
lessonParagraph(
  "HTTP Methods",
  "The most common HTTP methods are:\n\n- **GET** — retrieve\n- **POST** — create\n- **PUT** — replace\n- **DELETE** — remove\n\nEach method has defined semantics.",
);
```

---

## Part 1 — The Question Bank DSL

### Element Structure (`CourseEl`)

```ts
interface CourseEl {
  tag: string;                    // Element type
  attr?: Record<string, unknown>; // Attributes
  kids?: (CourseEl | string)[];   // Children
  interaction?: Interaction;      // Only on question elements
}
```

Bank builders return `QuestionDefinition` (not `CourseEl`). The server wraps their content tree and stores them in the bank.

### Atoms (Primitives)

| Function | Purpose |
|----------|---------|
| `questionStem(text)` | Bold question prompt |
| `choiceLabel(text, id)` | Answer option |
| `choiceImage(src, alt, id)` | Image answer option |
| `feedbackCorrect(text)` | Green success message |
| `feedbackIncorrect(text)` | Red error message |
| `hintToggle(text)` | Collapsible hint |
| `sectionHeading(text, level?)` | Heading (default level 2) |
| `progressIndicator(current, total)` | Progress bar |
| `explanationText(text)` | Detailed explanation paragraph |
| `divider(margin?)` | Horizontal separator |

### Bank Question Molecules

First arg is always the bank-unique question `id` used by pool selectors. Last arg is an optional `BankQuestionOpts`:

```ts
interface BankQuestionOpts {
  scored?: boolean;         // default true (false for long-text & likert)
  retryable?: boolean;
  gating?: boolean;
  tags?: string[];          // used by byTagsSelector
  competencyIds?: string[]; // used by byCompetenciesSelector
  contentRefs?: Array<{ sectionId: string; anchorId?: string }>;
}
```

**`bankRadioQuestion(id, stem, choices, correctId, opts?)`**

```ts
bankRadioQuestion("q-http-meaning", "What does **HTTP** stand for?", [
  { id: "a", text: "HyperText Transfer Protocol" },
  { id: "b", text: "High Tech Transfer Protocol" },
], "a", { tags: ["basics", "terminology"] });
```

**`bankCheckboxQuestion(id, stem, choices, correctIds, opts?)`**

```ts
bankCheckboxQuestion("q-methods", "Which are valid HTTP methods?", [
  { id: "a", text: "GET" }, { id: "b", text: "POST" },
  { id: "c", text: "FETCH" }, { id: "d", text: "DELETE" },
], ["a", "b", "d"]);
```

**`bankTrueFalseQuestion(id, stem, correctAnswer, opts?)`**

```ts
bankTrueFalseQuestion("q-stateless", "HTTP is stateless.", true);
```

**`bankTextInputQuestion(id, stem, correctAnswer, opts?)`** — short free text

```ts
bankTextInputQuestion("q-404", "What status code means 'Not Found'?", "404", {
  placeholder: "Enter status code…",
});
```

**`bankLongTextQuestion(id, stem, opts?)`** — essay. Unscored by default.

```ts
bankLongTextQuestion("q-essay", "Explain HTTP:", { rows: 6, maxLength: 1000 });
```

**`bankNumericQuestion(id, stem, correctAnswer, opts?)`**

```ts
bankNumericQuestion("q-add", "What is 2 + 2?", 4, { min: 0, max: 100, step: 1 });
```

**`bankMatchingQuestion(id, stem, pairs, opts?)`**

```ts
bankMatchingQuestion("q-match", "Match each term to its definition:", [
  { leftId: "l1", leftText: "HTTP", rightId: "r1", rightText: "Web protocol" },
  { leftId: "l2", leftText: "DNS",  rightId: "r2", rightText: "Name lookup"  },
]);
```

**`bankSequencingQuestion(id, stem, items, correctOrder, opts?)`**

```ts
bankSequencingQuestion("q-lifecycle", "Order the HTTP lifecycle:", [
  { id: "a", text: "DNS" }, { id: "b", text: "TCP" },
  { id: "c", text: "HTTP request" }, { id: "d", text: "Response" },
], ["a", "b", "c", "d"]);
```

**`bankImageSelectionQuestion(id, stem, images, correctId, opts?)`**

```ts
bankImageSelectionQuestion("q-icon-db", "Which icon represents a database?", [
  { id: "img1", src: "/icons/db.png",     alt: "Database icon" },
  { id: "img2", src: "/icons/server.png", alt: "Server icon"   },
], "img1");
```

**`bankLikertQuestion(id, stem, scale, opts?)`** — rating, unscored by default

```ts
bankLikertQuestion("q-confidence", "How confident are you?", 5, {
  endLabels: ["Not confident", "Very confident"],
});
```

### Pool Selectors

| Function | Purpose |
|----------|---------|
| `explicitSelector(questionIds)` | Hand-picked list |
| `byTagsSelector(tags, match?)` | Match by tag; `match: "any"` (default) or `"all"` |
| `byCompetenciesSelector(competencyIds, minProficiency?)` | Questions tagged with these competencies |
| `compoundSelector(selectors)` | AND of sub-selectors |

### `pool(id, name, selector)` and `questionBank(meta, body)`

```ts
import {
  questionBank, pool, byTagsSelector, explicitSelector,
  bankRadioQuestion, bankTrueFalseQuestion,
} from "@course/components";

export default questionBank(
  { name: "HTTP Basics Bank", description: "Baseline HTTP knowledge" },
  {
    questions: [
      bankRadioQuestion("q-meaning", "What does HTTP stand for?", [...], "a",
        { tags: ["basics"] }),
      bankTrueFalseQuestion("q-stateless", "HTTP is stateless.", true,
        { tags: ["basics"] }),
    ],
    pools: [
      pool("basics-pool", "All basics", byTagsSelector(["basics"])),
      pool("finale-pool", "Finale",      explicitSelector(["q-meaning", "q-stateless"])),
    ],
  },
);
```

---

## Part 2 — The Course DSL

### Organisms

**`lessonSection(id, title, contentEls, opts?)`** — teaching section (default role `learning`)

```ts
lessonSection("lesson-intro", "Introduction to HTTP", [
  lessonParagraph("What is HTTP?", "HTTP is the foundation of the web…"),
  lessonParagraph("How it Works",  "When you visit a URL…"),
]);
```

`opts`:
- `prerequisites?: string[]`
- `role?: string` — override preset (`learning`, `practice`, `graded`)
- `overrides?: SectionRoleOverrides` — tweak individual behavior properties

**`assessmentSection(id, title, contentEls, opts?)`** — assessment section (default role `graded`)

```ts
assessmentSection("assess-basics", "HTTP Knowledge Check", [
  questionSlot("warmup", { pool: "basics-pool", count: 3, ordering: "random" }),
  questionSlot("finale", { pool: "finale-pool", count: 2, ordering: "fixed"  }),
], { prerequisites: ["lesson-intro"], passingScore: 0.7 });
```

Adds `passingScore?: number` — switches completion rule to score-threshold.

**`questionSlot(slotId, rule)`** — atom that the server resolves at runtime

```ts
interface SelectionRule {
  pool: string | string[];   // pool ID, or ordered list unioned before selecting
  count: number;             // how many questions to draw
  ordering: "fixed" | "random";
  seed?: string;             // override deterministic seed
}
```

**`topic(id, title, sections, opts?)`**

```ts
topic("topic-http", "HTTP Fundamentals", [
  lessonSection(...),
  assessmentSection(...),
], { description: "Core HTTP concepts" });
```

**`course(meta, topics)`** — top-level compositor

```ts
course(
  {
    title: "Web Development Basics",
    description: "Learn the foundations.",
    version: "1.0",
    mode: "teaching",                     // "teaching" | "assessment"
    passingScore: 0.7,
    competencyFrameworkId: "fw_abc123",   // optional
  },
  [ topic("topic-http", "HTTP", [...]) ],
);
```

### Content Molecules

| Function | Purpose |
|----------|---------|
| `lessonParagraph(heading, body)` | Teaching content block (markdown-aware) |
| `mediaBlock(src, type, caption?)` | Image/video/audio (`type: "image" \| "video" \| "audio"`) |

### Section Roles

Every section has a `roleKey` controlling six behavioral properties:

| Role | `immediateFeedback` | `stripAnswerKeys` | `scoringMode` | Typical use |
|------|---------------------|-------------------|---------------|-------------|
| `learning` | true | true | none | Default for `lessonSection` |
| `practice` | true | false | formative | Low-stakes drills |
| `graded`   | false | true | summative | Default for `assessmentSection` |

### Cross-References Between Sections

Author-supplied ids (e.g. `"assess-finale"` in `assessmentSection("assess-finale", ...)`) are **local symbols only**. They exist for you to wire one section to another at authoring time. The publish pipeline mints a fresh nanoid for every section and rewrites every cross-reference attr through that map — the literal string you wrote does **not** appear in the runtime payload.

Currently rewritten attrs:

- `prerequisites: ["lesson-intro"]` on any section opts
- `<section-results sectionId="assess-finale" />` (the `sectionId` attr on a `section-results` element inside another section's content)

What this means in practice:

- Use author ids freely to refer to sibling sections — `prerequisites`, `<section-results>`, and any future cross-ref attr the platform adds will resolve them correctly.
- Don't expect the author id to be the runtime id. If you need to address a section programmatically (e.g. via xAPI), consume the ids returned in the publish response.
- If you author a new attr that contains a section id and the platform doesn't yet rewrite it, it will silently break at runtime. Adding new cross-ref-shaped attrs is a server change, not just a DSL change.

---

## Part 3 — Creating a Course: Step by Step

### 1. Author and POST the question bank

```
POST /t/{tenantSlug}/api/v1/question-banks
Authorization: Bearer course_...
Content-Type: application/json

{ "dsl": "<bank DSL script as a string>" }
```

Response (201):

```json
{ "id": "bnk_abc", "name": "HTTP Basics Bank", "questionCount": 8, "poolCount": 2 }
```

Pool IDs are the ones you declared (`basics-pool`), not generated — hard-code them in the course DSL.

### 2. Author and POST the course

```ts
export default course(
  { title: "Web Dev Basics", version: "1.0", mode: "teaching", passingScore: 0.7 },
  [
    topic("topic-http", "HTTP", [
      lessonSection("lesson-intro", "Intro", [
        lessonParagraph("Overview", "HTTP is how the web talks."),
      ]),
      assessmentSection("assess-intro", "Quiz", [
        questionSlot("warmup", { pool: "basics-pool", count: 2, ordering: "random" }),
      ], { prerequisites: ["lesson-intro"], passingScore: 0.7 }),
    ]),
  ],
);
```

```
POST /t/{tenantSlug}/api/v1/courses
Authorization: Bearer course_...
Content-Type: application/json

{ "dsl": "<course DSL script>", "slug": "web-dev-basics" }
```

Response (201):

```json
{
  "id": "crs_abc",
  "slug": "web-dev-basics",
  "url": "/c/{tenantSlug}/web-dev-basics",
  "fullUrl": "http://localhost:8789/c/{tenantSlug}/web-dev-basics"
}
```

Errors are shaped `{ "error": string, "phase": "validation" | "execution" | "persistence" }`.

### 3. Learner loads the course

The server walks each section, resolves every `questionSlot` against its pool, persists the picked question IDs to `slot_resolutions` keyed on `(registrationId, slotId)`, and streams a resolved tree to the renderer. Reloads hit the same resolutions so learners see stable questions.

---

## Aligning to a Competency Framework

1. Create or fetch a framework via `/t/{slug}/api/v1/competency-frameworks`.
2. Set `meta.competencyFrameworkId` on the course.
3. Tag questions in the **bank** with `competencyIds: [...]`.
4. Use `byCompetenciesSelector(["comp_1"])` in pools to target specific competencies.

At assessment time the renderer writes per-competency coverage into `assessmentResults.competencyCoverage`.

---

## Best Practices

- **Unique IDs everywhere** — Questions (bank-unique), pools (bank-unique), sections/topics/slots (course-unique). Use descriptive prefixes: `q-http-meaning`, `lesson-intro`, `assess-final`, `basics-pool`, `warmup-slot`.
- **Tag generously** — Tags on questions drive `byTagsSelector` pools. Plan a small taxonomy before authoring.
- **Pool organization** — A few broad pools by topic/competency plus a couple of `explicitSelector` pools for fixed finales is usually enough.
- **Mix question types** — Radio, checkbox, true/false, fill-in, etc.
- **Progressive difficulty** — Put drills in `role: "practice"` sections before `graded` assessments.
- **Prerequisites** — `prerequisites: ["lesson-id"]` on assessments forces completion order.
- **Passing scores** — 0.6–0.8 typical. Set on course meta; override per-assessment via `passingScore`.
- **Separate layers** — Sections describe layout + which pool to draw from. Banks describe question content. Never inline answer data in a section's content tree.

---

## Interaction Types Reference

| Type | xAPI Type | Correct Pattern Format |
|------|-----------|------------------------|
| Single choice | `choice` | `["a"]` |
| Multiple choice | `multiple-choice` | `["a[,]b[,]d"]` (sorted) |
| True/false | `true-false` | `["true"]` or `["false"]` |
| Fill-in | `fill-in` | exact text |
| Numeric | `numeric` | number as string |
| Long fill-in | `long-fill-in` | free text (typically unscored) |
| Matching | `matching` | `["a[.]1[,]b[.]2"]` |
| Sequencing | `sequencing` | `["a[,]b[,]c"]` |
| Likert | `likert` | scale value (typically unscored) |
| Image choice | `image-choice` | `["img1"]` |

The bank builders compute `correctResponsesPattern` for you.

## xAPI Verbs Emitted

- `initialized` — course loads
- `experienced` — section viewed
- `answered` — question submitted
- `completed` — all scored interactions answered
- `passed` / `failed` — score vs passing threshold
- `suspended` / `resumed` — tab blur/focus

## Imports

```ts
// Bank DSL
import {
  bankRadioQuestion, bankCheckboxQuestion, bankTrueFalseQuestion,
  bankTextInputQuestion, bankLongTextQuestion, bankNumericQuestion,
  bankMatchingQuestion, bankSequencingQuestion, bankImageSelectionQuestion,
  bankLikertQuestion,
  explicitSelector, byTagsSelector, byCompetenciesSelector, compoundSelector,
  pool, questionBank,
} from "@course/components";

// Course DSL
import {
  // Atoms
  questionStem, choiceLabel, choiceImage, feedbackCorrect, feedbackIncorrect,
  hintToggle, sectionHeading, progressIndicator, explanationText, divider,
  questionSlot,
  // Content molecules
  lessonParagraph, mediaBlock,
  // Organisms
  lessonSection, assessmentSection, courseHeader, courseFooter, scoreCard,
  topic, course,
} from "@course/components";

import type {
  CourseEl, Interaction, CourseDefinition, CourseSection, CourseTopic,
  SelectionRule, QuestionDefinition, QuestionPoolDefinition,
  QuestionBankDefinition, PoolSelector, SectionRoleOverrides,
  CompletionRule, StyleDef,
} from "@course/shared";
```
