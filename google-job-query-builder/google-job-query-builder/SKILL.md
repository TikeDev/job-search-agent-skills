---
name: google-job-query-builder
description: >
  Builds precise Google Boolean search queries for job hunting, complete with
  clickable search URLs, clause-by-clause explanations, and a curated canonical
  list of ATS platforms and job boards. Trigger this skill whenever the user
  wants to find jobs on Google, asks to "build a job search query", mentions
  Boolean job search, wants to search across job boards or ATS platforms, or
  says anything like "help me find jobs for X role", "generate a job search
  string", "how do I search for jobs on Google", or "make a Google search for
  jobs". Also trigger when the user describes a job they're looking for in
  natural language, even casually — e.g. "I'm looking for remote AI roles" or
  "help me find full stack jobs in Boston". Always use this skill proactively
  rather than building a query from scratch.
---

# Google Job Query Builder

Builds Google Boolean search queries for job hunting. Supports two interaction modes — freeform description or guided questionnaire — and outputs a raw query, a clickable Google URL, and a clause-by-clause explanation.

---

## Interaction Modes

### Mode A — Freeform
User describes what they're looking for in natural language. Extract parameters from their description, fill in defaults for anything missing, build the query, and present the output. Confirm any assumptions made.

Example trigger: *"I'm looking for remote mid-level AI engineer roles"*

### Mode B — Guided Questionnaire
Walk the user through parameters one group at a time. Don't dump all questions at once — ask in 2-3 logical batches:

**Batch 1 — Role**
- Job title(s) / function (e.g. engineer, product manager, designer)
- Seniority level(s) (see Seniority section below)
- Domain/specialty keywords (e.g. AI, audio, full stack, backend)

**Batch 2 — Location**
- Remote, onsite, hybrid, or flexible?
- If onsite/hybrid: city, metro area, state/country
- Any location disambiguation needed (e.g. "Cambridge" — UK or MA?)

**Batch 3 — Targeting**
- Specific companies to target? (adds `intitle:` or company name terms)
- Anything to exclude? (titles, companies, keywords)
- Date range? (after:, before:, or a window like "posted in the last 2 weeks")
- Which job sites to search? (default: full canonical list — see references/sites.md)
- How strict should the query be? (broad / balanced / strict — affects use of AND vs OR and phrase quoting)

---

## Query Construction Rules

### Operators to use
- `"phrase"` — exact phrase match (use for multi-word titles and location names)
- `|` or `OR` — alternatives within a group
- `AND` — require both groups to match (implicit between groups, explicit for clarity)
- `-term` — exclude a word (no space after dash)
- `site:domain.com` — restrict to a specific domain
- `after:YYYY-MM-DD` / `before:YYYY-MM-DD` — date filtering
- `( )` — group alternatives

### Query structure (in order)
```
[LOCATION] AND [DOMAIN/SPECIALTY] AND [FUNCTION] [SENIORITY EXCLUSIONS] [DATE] [SITES]
```

### Location block
- Remote roles: `(remote AND "United States")` or `(remote AND US)` — adjust country as needed
- City/metro: use pipe-separated alternatives with quoted multi-word names
  - Example: `(Boston | Massachusetts | Somerville | "Cambridge, MA")`
  - Always quote ambiguous place names: `"Cambridge, MA"` not `Cambridge`
  - Never use bare `Cambridge` — add `, MA` or `-"United Kingdom"` to disambiguate

### Domain/specialty block
- Use pipe-separated alternatives with quoted multi-word phrases
- Example: `(ai | "machine learning" | agentic | "ai-native" | "ai-enabled")`
- For full stack: `(fullstack | "full stack" | "full-stack" | frontend | backend)`

### Function block
- `(engineer | developer | builder)` for IC engineering roles
- `("product manager" | PM)` for PM roles
- `(designer | "UX engineer")` for design roles

### Seniority exclusions
Based on target level, exclude titles above and below:

| Target level | Exclude |
|---|---|
| Junior / Entry | *(no exclusions, or add `-senior` if needed)* |
| Mid-level | `-junior -associate -staff -lead -principal -director -VP -"vice president"` |
| Senior | `-junior -associate -staff -principal -director -VP -"vice president"` |
| Mid or Senior | `-junior -associate -staff -principal -director -VP` |
| Senior+ / Staff | `-junior -associate` |

Always use `-` exclusions, not `NOT` (Google treats `NOT` unreliably).

### Strictness levels
- **Broad**: Use `|` liberally, fewer required AND groups, shorter exclusion list
- **Balanced** (default): AND between major groups, OR within groups, standard exclusions
- **Strict**: Wrap key phrases in quotes, use AND between more groups, longer exclusion list

### Date filtering
- After a date: append `after:YYYY-MM-DD`
- Before a date: append `before:YYYY-MM-DD`
- Within a window: combine both, e.g. `after:2026-06-01 before:2026-06-13`
- "Last N days": calculate from today's date

### Site block
See `references/sites.md` for the full canonical list and categories.

Default behavior: include the full canonical list unless the user asks to narrow it.
Present as a parenthesized `site:` OR group at the end of the query.

---

## Output Format

Always produce all three of the following:

### 1. Raw query
```
[query string here]
```

### 2. Clickable Google URL
URL-encode the query and prepend `https://www.google.com/search?q=`
Format as a Markdown link: `[Search Google →](https://www.google.com/search?q=...)`

To URL-encode:
- Spaces → `+`
- `"` → `%22`
- `(` → `%28`, `)` → `%29`
- `|` → `%7C`
- `:` → `%3A`
- `-` → `-` (no encoding needed)
- `*` → `%2A`

### 3. Clause explanation
Brief bullet list explaining each logical group in plain English. Don't show the clause itself — just describe what it does in a natural sentence. Prefix each bullet with a relevant emoji. Only include categories that are actually used in the query.

Emoji and phrasing conventions per category:
- 📍 **Location:** Describe the geography; note any disambiguation (e.g. Cambridge quoted to avoid UK results)
- 🧠 **Specialty:** Describe the domain/focus area and any variant coverage
- 🛠️ **Function:** Describe the role type (e.g. IC engineering roles only)
- 🪜 **Seniority:** Use "No X, Y, Z titles" phrasing — e.g. "No junior, associate, staff, principal, or director titles"
- 🚫 **Company exclusions:** Use "No results from X, Y, Z" phrasing
- 📅 **Date range:** Use "Posted [Month Day] – [Month Day, Year]" format — e.g. "Posted May 1 – June 13, 2026". For open-ended: "Posted after May 1, 2026"
- 🌐 **Sites:** Describe which site categories are included/excluded and why

---

## Batch Output Format

When the user requests multiple queries at once (e.g. "give me queries for remote AI roles, Boston full stack roles, and PM roles at fintech companies"), present them in an organized layout:

### Structure
Use a numbered header per query, with a short descriptive label:

---

#### Query 1 — [Short Label, e.g. "Remote AI Engineer"]

**Raw query:**
```
[query]
```
[Search Google →](url)

- **Location:** ...
- **Specialty:** ...
- **Exclusions:** ...
- **Sites:** ...

---

#### Query 2 — [Short Label]

...and so on.

---

### Shared assumptions block
If multiple queries share the same assumptions (e.g. same site list, same seniority exclusions), state them once at the top rather than repeating per query:

> *All queries below use: mid/senior seniority exclusions, full canonical site list, no date filter. Per-query differences noted inline.*

### Diff notes
For queries that deviate from the shared assumptions, call it out inline under that query's clause explanation rather than repeating the full assumptions block.

---

## Assumptions to Confirm

After building from freeform input, always state what you assumed and offer to adjust:
> *Assumed: mid/senior level (excluded staff, lead, principal, director). Used full canonical site list. No date filter. Let me know what to change.*

---

## Reference Files

- `references/sites.md` — Canonical list of ATS platforms and job boards, organized by category, with notes on reliability
