# Zoning: when one project outgrows core/state

Most projects are fine with `core.md` + `state.md`. A few are not.

## The symptom

A project that is really **several jobs wearing one name**. Its `state.md`
becomes a list of unrelated statuses, and loading it to answer one question drags
in four other topics. The 30-day test stops helping, because everything in the
file passes it — just for different reasons.

Concretely, you have outgrown core/state when:

- `state.md` has 4+ top-level sections that never reference each other
- Two sections have different owners (you drive one, someone external drives another)
- Opening the project to do one thing means reading three things you will ignore
- You catch yourself asking *"which part of this project do you mean?"*

## The fix: single responsibility per zone

Split the project into zones, each owning **one** responsibility, each with its
own `state.md`:

```
memory/projects/a1_example/
├── core.md              ← the router + the permanent layer
├── facts/               ← 💎 verified facts every zone needs
│   └── profile.md
├── inbox/               ← 📥 incoming external material
│   └── from_agency.md
├── selection/           ← 🏫 zone: choosing
│   └── state.md
├── documents/           ← 📄 zone: writing
│   └── state.md
└── people/              ← 🤝 zone: contacting
    └── state.md
```

`core.md` becomes the **entry point and router**: it holds the permanent layer
*and* a table mapping each zone to its files.

## The three zoning rules

### 1. Facts sink

A verified fact — a date, a number, a confirmed policy — lives in `facts/`,
**once**. Zones read from it. The moment the same fact exists in two zones, they
will disagree, and you will not notice which one is stale.

### 2. State belongs to whoever drives it

Not to whoever mentioned it. If an external party is chasing something, its
status lives in the zone that owns that relationship — even if you first heard
about it while working in a different zone.

### 3. Cross-zone links only, never copies

Reference `[[the-other-file]]`. Never duplicate the content. A link that goes
stale is visible; a copy that goes stale is invisible.

## Routing behaviour

When the user opens a zoned project, the assistant **lists the zones and lets
them pick**, then loads only that zone:

```
A1 has 4 zones — pick one and I'll load only that:
🏫 Selection · 📄 Documents · 🤝 People · 💎 Facts
```

🚨 **Never** ask a vague "what do you want to do?" for a zoned project. The whole
point is that the user does not have to hold the structure in their head.

🚨 **Sorting is the assistant's job.** If the user asks a documents question
while "in" the selection zone, do not correct them. Load what is needed, and note
`📌 filed under Documents` in one line when writing it down. Correcting a user's
filing is a tax on thinking out loud — and thinking out loud is what the system
exists to support.

## Two special axes

Large projects often have content that is not a zone but cuts across all of them:

- **📐 Spec / constitution** — the requirements every decision must satisfy. When
  zones conflict, this settles it. Lives in `core.md` or its own file.
- **🪨 Anchor** — for projects with an emotional dimension (a career change, a
  long exam, a health goal): what to re-read when the user is discouraged.
  Opening it means **companion mode, not work mode** — no TODOs, no next actions.

The anchor file sounds soft. It is the most-used file in a two-year project.

## When *not* to zone

Zoning costs navigation overhead. Do not do it because a project feels
important — do it when `state.md` genuinely holds unrelated jobs. Two zones is
usually not worth it; four or more usually is. Under 5 K tokens, stay with
core/state.
