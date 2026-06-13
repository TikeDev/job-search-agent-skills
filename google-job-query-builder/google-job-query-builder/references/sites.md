# Canonical Job Site List

Default: include all sites below unless user asks to narrow by category or exclude specific ones.
Present as: `(site:domain1 | site:domain2 | ...)`

---

## ATS Platforms
*Companies post jobs directly here. High signal, low noise.*

| Domain | Notes |
|---|---|
| `site:jobs.greenhouse.io` | Very common at mid-size tech companies |
| `site:boards.greenhouse.io` | Alternate Greenhouse subdomain |
| `site:myworkdayjobs.com` | Enterprise/large company standard |
| `site:jobs.lever.co` | Popular at startups and growth-stage |
| `site:jobs.ashbyhq.com` | Common at YC/VC-backed startups |
| `site:app.ashbyhq.com` | Alternate Ashby subdomain |
| `site:smartrecruiters.com` | Used by mid-to-large companies |
| `site:workable.com` | Used by small/mid-size companies |
| `site:jobs.jobvite.com` | Older ATS, still common at established companies |
| `site:recruiting.paylocity.com` | Common at mid-size non-tech companies |
| `site:careers.icims.com` | Enterprise ATS |
| `site:apply.workable.com` | Alternate Workable subdomain |

---

## Startup / Tech-Focused Boards
*Good for VC-backed, early-stage, and tech-forward companies.*

| Domain | Notes |
|---|---|
| `site:wellfound.com` | Formerly AngelList Talent / AngelList; startup-focused |
| `site:ycombinator.com/jobs` | YC company jobs |
| `site:builtin.com` | Tech company jobs, good Boston/NYC/SF coverage |
| `site:jobright.ai` | AI-curated job board, good for tech roles |
| `site:techjobsforgood.com` | Mission-driven tech roles |

---

## Remote-Focused Boards
*Include when searching remote roles; can omit for local-only searches.*

| Domain | Notes |
|---|---|
| `site:remotive.com` | Curated remote tech jobs |
| `site:remoteok.com` | High volume remote board |
| `site:weworkremotely.com` | Popular remote board, varies in quality |
| `site:remote.co` | Curated remote listings |

---

## General / Aggregator Boards
*Higher noise but broad coverage.*

| Domain | Notes |
|---|---|
| `site:dice.com` | Tech-focused, skews toward contract roles |
| `site:simplyhired.com` | Aggregator; now partnered with Indeed |

---

## On-Request Only
*These sites have unreliable Google indexing or high noise. Only include if user asks.*

| Domain | Notes |
|---|---|
| `site:linkedin.com/jobs` | Poorly indexed by Google; better searched natively on LinkedIn |
| `site:indeed.com` | Google often deindexes Indeed; hit-or-miss |
| `site:ziprecruiter.com` | Lower signal-to-noise ratio |
| `site:monster.com` | Mostly legacy roles |
| `site:glassdoor.com/job` | Inconsistent indexing |

---

## Category Groups for Quick Selection

When the user wants to narrow the site list, offer these presets:

- **ATS only** — Most direct company postings, least noise
- **ATS + Startup boards** — Good default for tech roles
- **ATS + Startup + Remote boards** — Best for remote job searches
- **Full canonical** (default) — All except on-request-only sites
- **Custom** — User picks individual sites or adds their own
