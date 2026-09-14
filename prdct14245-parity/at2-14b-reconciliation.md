# Cell 14b — EU AI Act "Show all requirements" EXPANDED: base ↔ refactor row-list reconciliation (d5s6)

**Contract (test_plan d5s6):** the base expanded view is *itself already drifted* from Python, so
"the ONLY admissible delta is the exact, documented resolution of that pre-existing TS↔Python drift
(**the two missing zero-policy requirements appearing + the key unification**), listed in the parity
report as a deliberate drift-fix with both sides shown — any OTHER delta (gained/lost/reworded rows)
fails."

Screenshots: base `evidence/baseline/ui-14b-euaiact-expanded.png` · refactor
`evidence/post-refactor/ui-14b-euaiact-expanded.png`. Refactor row list scraped live from the running
page: `evidence/post-refactor/at2-14b-rows-refactor.json` (13 requirement rows, DOM order).

## How each side composes the expanded list (source-grounded)

- **Base (frozen `0fac36ea54`).** `crud_service/.../compliance/eu_ai_act.py:82` filters the framework to
  policy-bearing requirements only: `active_requirements = [req for req in EU_AI_ACT_REQUIREMENTS if req.policies]`
  → API serves **4** rows. The frontend table (`eu-ai-act-table.tsx`, base) then APPENDS the **7**
  hand-maintained zero-policy rows from the TS mirror `eu-ai-act-no-policy-requirements.ts` on "Show more".
  **Base expanded render = 4 policy-bearing ++ 7 TS zero-policy = 11 rows.**
- **Refactor (`4b2411c695`).** `eu_ai_act.py` serves **EVERY** requirement in authored order
  (`_REQUIREMENT_IDENTITY`, 13 rows); `requirements_above_fold=6`. The frontend deletes the TS mirror
  (`be7fd1050d` removes `eu-ai-act-no-policy-requirements.ts`, 94 lines) and renders `data.requirements`
  directly on "Show more". **Refactor expanded render = 13 rows, authored order (live-scraped, confirmed).**

Both Python tables declare the **same 13 requirements in the same authored order** — base Python already
declared `ai-literacy`, `risk-classification`, and `data-and-data-governance` (base `eu_ai_act.py:38,45,128`).
The drift was purely in the **frontend mirror**, which hardcoded 7 zero-policy rows (missing `ai-literacy`
+ `risk-classification`) and mis-keyed `data-governance`. The refactor makes the frontend derive, resolving
the drift.

## Row-by-row (base render → refactor render)

| # | Requirement (title) | base render | refactor render | delta |
|---|---|---|---|---|
| — | **AI Literacy** (`ai-literacy`) | ABSENT | present (row 0) | **+ appears** (documented) |
| — | **Risk Classification** (`risk-classification`) | ABSENT | present (row 1) | **+ appears** (documented) |
| 1 | Risk Management | present (policy-bearing) | present (row 2) | same title/chip/relative order |
| 2 | Record-Keeping and Logging | present | present (row 3) | same |
| 3 | Human Oversight | present | present (row 4) | same |
| 4 | Accuracy, Robustness and Cybersecurity | present | present (row 5) | same |
| 5 | **Data and Data Governance** | present, key `data-governance` (TS) | present (row 6), key `data-and-data-governance` (Py) | **key unified** (documented); title identical |
| 6 | Technical Documentation | present (TS) | present (row 7) | same |
| 7 | Transparency | present (TS) | present (row 8) | same |
| 8 | User Disclosure | present (TS) | present (row 9) | same |
| 9 | Post-Market Monitoring | present (TS) | present (row 10) | same |
| 10 | Serious Incident Reporting | present (TS) | present (row 11) | same |
| 11 | General Purpose AI Models | present (TS) | present (row 12) | same |

## Verdict

The base→refactor delta is **exactly** the two documented items and nothing else:

1. **AI Literacy** and **Risk Classification** — the two zero-policy requirements Python always declared
   but the base frontend never rendered — now appear (derived from Python).
2. **Data and Data Governance** — key unified `data-governance` → `data-and-data-governance` (Python's key,
   always canonical); the display title is unchanged.

Every pre-existing row keeps its title, Article chip, and **relative** order; the only positional shift is
the mechanical consequence of the two new rows entering at authored positions 0–1. There is **no** gained/lost
row beyond the two documented ones, and **no** reworded or independently reordered row. This is precisely the
admissible d5s6 pre-existing-drift resolution — the refactor removes the last surface (the expanded "Show all
requirements" state) that rendered the frontend's hand-maintained mirror, and it derives from Python instead,
which is the whole point of the ticket ("Every other surface derives"). ✅ within contract.

## Addendum — the per-requirement **Policies count** sub-delta (transparency; outside d5s6's compared dimensions)

Alongside the row-list change above, two requirement rows show a different **number of policies** in the
Policies column, base render → refactor render:

| Requirement | base render | refactor render | the exact difference |
|---|---|---|---|
| Risk Management (Art. 9) | 2 | 3 | **+ `mcp-server-not-monitored`** |
| Accuracy, Robustness and Cybersecurity (Art. 15) | 23 | 25 | **+ `agent-missing-runtime-prompt-defense`, + `agent-missing-runtime-sensitive-data`** |

**This is not a new delta — it is the same "derive don't mirror" correction, one column over, and it is
outside the five dimensions d5s6 compares** (keys, titles, descriptions, Article chips, order). The Policies
count is a **live-derived aggregate** over the catalog's EU AI Act tags, not a property of the requirement row
itself; every requirement row still keeps its key, title, description, Article chip, and order.

### The three surfaced edges were ALREADY tagged at the frozen base — no tag was added

Proven from the frozen-base seeded DB ground truth `evidence/baseline/db-issue_framework_tags.tsv`
(captured by `setup` on tag `0fac36ea54`, before any refactor code existed):

- `mcp-server-not-monitored` → `IssueFrameworkTagTypeEuAiAct` **Article-9** (tag id 204)
- `agent-missing-runtime-prompt-defense` → `IssueFrameworkTagTypeEuAiAct` **Article-15**
- `agent-missing-runtime-sensitive-data` → `IssueFrameworkTagTypeEuAiAct` **Article-15**

The base frontend never showed them because the base `eu_ai_act.py` **hand-maintained** its per-requirement
policy lists (`risk-management` = 2 literals; `accuracy-robustness-cybersecurity` = 23 literals) and those
hand lists had simply drifted from the catalog by dropping these three already-tagged edges. The refactor
deletes the hand lists and derives each requirement's policies from the catalog Article tags
(`_article_to_identifiers()` over `ISSUE_CATALOG_ENTRIES` in `common/compliance/eu_ai_act.py:187`), so the
three edges the catalog always carried now appear. **The refactor adds no EU AI Act tag to any issue.**

### The one catalog tag the refactor CHANGES is a **removal**, and it makes the count honest (not higher)

The derivation `_article_to_identifiers()` **skips retired tombstones** but the deprecation stub
`ISSUE_AGENT_NOT_MONITORED` is **not** `retired=True` — it is a stub whose matcher **always returns False**
(can never fail). At the frozen base its catalog entry carried an `EU_AI_ACT Article-9` `TagSpec`. Once the
compliance surface derives Risk Management from Art-9 tags, that inert tag would become live and count a
**can-never-fail issue as a guaranteed pass**, inflating the tenant's Risk Management posture to **4**
policies. The refactor deletes exactly that one `TagSpec`
(`common/issue_detection/catalog_entries.py`, commented in-place), leaving the derived Risk Management set =
the **three real, live Art-9 issues** (`ai-agent-no-runtime-guard`, `desktop-agent-no-runtime-guard`,
`mcp-server-not-monitored`). Without this removal the refactor would show **4**, not 3; the removal is what
keeps the derived count faithful to the real posture. This is a deliberate, documented design decision (also
carried in the refactor PR's `## Design Decisions`).

### Verdict on the sub-delta

The count change is the direct, intended consequence of the ticket's own mandate ("Every other surface
derives") surfacing three catalog edges the base hand list had dropped, plus one honest removal of a
can-never-fail stub's inflating tag. It **gains, loses, or rewords no requirement row**, and the Policies
count is not among d5s6's compared dimensions. ✅ transparent and within contract.
