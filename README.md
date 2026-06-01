This skill was created by Claude using the following prompts:
* Create a 'Privacy by Design' skill. That skill should ensure that any software built is minimally invasive for its intended purpose. That means that data is not created, collected, used or shared if not absolutely necessary.  To see examples of what  to do, see privacypatterns.org. See also https://instituteofprivacydesign.org/ for some privacy standards and assurance criteria. Privacy anti-patterns (if you find any) are examples of what not to do. Include security patterns (for what to do) and anti-patterns (for what not to do). All security problems are also privacy problems, because they allow data access for unauthorized parties.  That said,  there are also privacy problems that are not security problems (e.g., unauthorized use of data for which the party has authorized access).
* Move the 'What you couldn't review' above 'Recommended next steps'.  'Recommended next steps' should always be last. And, if there are potentially other steps that could have been recommended - then tell the user to make the recommended changes and then rerun - or provide an option to get a complete list of all (not just the top 5) recommended next steps.
* If there are other laws that can be included in the analysis, then add them - but ask the user for relevant jurisdictions (defaulting to all) so the user can limit the analysis if desired.
* pull the regional tables out into references/regulations-by-jurisdiction.md and leave just a 10-row "most-cited" table in the SKILL.md with a pointer
* When I try to save the skill, I get this message: 'field 'description' in SKILL.md must be at most 1024 characters'
* How does this skill compare with other tools that detect privacy problems?

To which, Claude responded with:
Good question to ask before locking the skill in. Let me check the current tool landscape so my comparison reflects what's out there now, not just what I know from training.The privacy detection space sorts into about six categories. This skill overlaps with several of them but targets a slice most don't reach.

| Category | Examples | What they catch well | What they miss |
|---|---|---|---|
| **Static privacy-aware SAST** | Bearer (Cycode), Semgrep with privacy rules, CodeQL | Known patterns at PR/CI time — data-flow analysis that tracks sensitive data through code, OWASP Top 10 coverage, leakage through cookies/loggers/third-party analytics. Deterministic, scriptable, runs every build. | Whether collecting the data was *justified*; intent and product purpose; pure privacy failures that aren't structural code smells; novel patterns not in the rule library. |
| **Data discovery / DSPM** | BigID, Securiti, Cyera, Strac, AWS Macie, Open Raven, Concentric, OvalEdge | Where sensitive data actually lives across databases, lakes, SaaS, and cloud; classification, lineage, access analysis, and continuous monitoring tied to ownership and workflows. The "what do we actually have?" question. | Whether you should have collected it; what your API and frontend will do with it tomorrow; design-time decisions made before any data exists. |
| **Privacy posture / GRC** | OneTrust, TrustArc, DataGrail, Osano | ROPA inventory, consent and cookie management, DSAR workflows, vendor risk register, regulatory mapping, audit reports. | Code-level review; specific remediations engineers can ship; novel design issues; PR-time feedback loops. |
| **Privacy threat modeling** | LINDDUN methodology, Microsoft Threat Modeling Tool, IriusRisk | Privacy threats surfaced systematically at design time with a structured taxonomy. | Code-level findings; takes hours-to-days per session; typically done once and rarely re-run; doesn't tie back to specific code. |
| **Tracker / cookie scanners** | Blacklight (The Markup), Webbkoll, OneTrust Cookie Scanner | Real third-party scripts loading on the live site, cookies actually set, fingerprinting attempts on real visits. | Backend; first-party flows; API and schema design; everything not visible to a headless browser. |
| **PII detection / redaction for AI pipelines** | OpenAI Privacy Filter (open-weight model for context-aware PII detection in unstructured text), Microsoft Presidio, Private AI | Scrubbing PII from prompts, logs, and training data; a runtime guard between data and LLM APIs. | Anything outside the LLM-pipeline use case; design and consent questions; non-PII sensitive categories. |

### Where this skill is similar — and where it's different

This skill is structurally closest to privacy threat modeling and to general LLM code review, but it differentiates in three specific ways:

**It distinguishes security failures from pure privacy failures.** Static scanners are excellent at the first category — IDOR, mass assignment, hardcoded secrets, PII flowing to logs. The skill's "Category 2" (data going to an authorised party for an unconsented purpose: God View tools, purpose creep, linkable identifiers, sensitive content streamed to analytics vendors, ML training on data collected for unrelated purposes) is largely invisible to static analysis because *the data is in the expected place* — it's just being used in unexpected ways. Bearer's data-flow analysis can flag a sensitive field flowing to a third-party endpoint, which is real privacy value, but it can't tell you that a Registration Vacuum exists or that your "soft delete" is a Purpose Creep dressed up as deletion. Compliance platforms might catch these in a policy audit; they rarely catch them in code review.

**Named-pattern vocabulary.** Findings cite patterns and anti-patterns from privacypatterns.org (Selective Disclosure, Pseudonymous Identity, Layered Policy, Sticky Policies) and the security canon (Mass Assignment, Broken Object-Level Authorisation), with real-world precedent (BIPA settlements, FTC actions against GoodRx and BetterHelp, the Uber God View incident). The vocabulary matters — "you have a Registration Vacuum" is more triageable than "you're collecting too much."

**Jurisdictional scoping.** The skill asks (or infers) which regulations are in scope before producing findings and only cites in-scope regulations, drawing from the inline top-10 table plus the regional reference file. Most SAST tools either ignore regulation entirely or default to a single regime, usually GDPR.

### Honest weaknesses

- **Not deterministic.** Two runs may surface different findings. Not a CI gate. Static scanners give you the same answer every time; this one doesn't.
- **Bounded by the context window.** Sees only what's pasted in. Can't crawl a whole repo. Bearer or Cycode can.
- **No runtime visibility.** Tells you what the code *suggests* will be sent to Mixpanel; can't see what's actually transmitted on the wire. Tracker scanners can.
- **Citations need verification.** LLMs misremember article numbers, case names, effective dates. Treat references as starting points, not authoritative.
- **No automatic ticketing or dashboard.** Findings come as structured prose; turning them into Jira tickets is manual.
- **Coverage proportional to what's shared.** A 100-line snippet gets a 100-line review. A whole-codebase view needs a different category of tool.

### How it fits in a real privacy program

Honestly: as the **design-review and code-review layer** in a multi-layered stack, not a replacement for the others.

- A **DSPM/data-discovery tool** for the "what do we have?" question across cloud and SaaS.
- A **static scanner with data-flow analysis** (Bearer, Semgrep) for known patterns at CI time — the deterministic floor.
- A **GRC platform** if the org has audit and DSAR obligations.
- A **tracker scanner** for the live web property.
- A **PII redactor** like OpenAI Privacy Filter or Microsoft Presidio if AI pipelines handle sensitive prompts.
- **This skill** for design review, schema review, PR review — and especially for issues that are technically working but ethically or legally questionable. That's the gap the others leave.

A team that can only afford one privacy capability shouldn't pick this skill — they need data discovery and consent management first because compliance has hard requirements with statutory teeth. A team that already has those, and is asking "how do we catch the next privacy bug *before* it ships, not just inventory the ones we've already shipped?" — that's exactly where the skill earns its keep.

If you want, I can probe a couple of these tools more deeply (especially Bearer, since it's open-source and the closest direct comparison) and check whether there are anti-patterns it covers that the skill is missing, or vice versa.

* So I said: 'probe the tools'. Claude did, and came back with this:

Good — let me actually look at what these tools catch, not just describe them in the abstract. I'll go deepest on Bearer (open-source rules are inspectable), then look at the canonical privacy frameworks (OWASP, LINDDUN), then see what's missing from the skill.Now let me check the OWASP Top 10 Privacy Risks framework — it's the canonical privacy-specific OWASP list (distinct from the security Top 10) and a common benchmark:Probed. Here's what I found and what it implies for the skill.

## What each tool actually catches

**Bearer (Cycode)** — open-source SAST with privacy-aware data-flow analysis. Currently 473 rules across Ruby, JS/TypeScript, Java, PHP, Go, Python. The rule taxonomy falls into clear buckets:

- **Standard security SAST** (most of the rules) — SQL/NoSQL injection, XSS, SSRF, deserialisation, weak crypto, hardcoded secrets, weak random, missing TLS, CSRF, path traversal, LDAP injection, XXE, command injection. These map cleanly to OWASP Top 10 + CWE.
- **Sensitive-data leakage to outputs** — PII in logger messages, mapped to CWE-532 (Insertion of Sensitive Information into Log File) and OWASP A09; PII in exception messages; PII in HTTP GET parameters because query strings get cached in server logs and browser history; PII in stdout; PII in cookies.
- **Named-third-party leakage** — and this is the interesting part. Bearer ships rules for specific vendor SDKs: Airbrake, Algolia, BigQuery, Bugsnag, ClickHouse, Datadog, ElasticSearch, Google Analytics, Google Dataflow, Honeybadger, New Relic, OpenTelemetry, RollBar, Segment, Sentry. If a tagged "sensitive" data type flows into a call to one of those, it fires. All are categorised CWE-201 / OWASP A01.
- **Cookie/session hygiene** — HttpOnly, Secure, SameSite, session storage.
- **Crypto hygiene** — weak ciphers, weak modes (ECB, padded CBC), weak hashing for passwords, weak key sizes, missing certificate verification.

Custom rules are supported, but the shipped rules are the canonical answer to "what Bearer catches."

**OWASP Top 10 Privacy Risks v2.0** (2021) is the privacy-specific OWASP framework — separate from the security Top 10. Risks are tagged as organisational, technical, or both, and the list draws on the OECD Privacy Guidelines as a framework:

| # | Risk | Type | Skill coverage |
|---|---|---|---|
| P1 | Web Application Vulnerabilities | Tech | ✅ Covered (Security Category 1, OWASP-A* anti-patterns) |
| P2 | Operator-sided Data Leakage — loss of confidentiality from insufficient access controls, insecure storage, or lack of awareness | Both | ✅ Covered (God View, internal admin lens) |
| P3 | Insufficient Data Breach Response — failing to notify affected data subjects or remedy the cause | Both | ⚠️ Mentioned only briefly (Data Breach Notification Pattern); no dedicated lens |
| P4 | Consent on Everything — aggregated consent rather than separate consent per purpose | Org | ✅ Covered (Bundled Consent anti-pattern, Lawful Consent pattern) |
| P5 | Non-transparent Policies, Terms and Conditions — failure to make processing easy to understand for non-lawyers | Org | ✅ Covered (Layered Policy Design, Minimal Information Asymmetry) |
| P6 | Insufficient Deletion of Personal Data | Both | ✅ Covered (Permanent Storage, Backup Immortality) |
| P7 | Insufficient Data Quality — using outdated, incorrect, or bogus user data | Both | ❌ **Not covered at all** |
| P8 | Missing or insufficient session expiration — may result in collection of additional user data without awareness | Tech | ✅ Covered (Weak Session Management) |
| P9 | Inability of users to access and modify their data | Both | ✅ Covered (Privacy Dashboard, Personal Data Table) |
| P10 | Collection of data not required for the user-consented purpose | Org | ✅ Covered (Registration Vacuum, Collect Now Decide Later) |

**LINDDUN** — already in the skill.

**Microsoft Presidio / OpenAI Privacy Filter / Private AI** — runtime PII detection in unstructured text. OpenAI's Privacy Filter, released April 2026, is an open-weight model for context-aware PII detection that can run locally and process long inputs in a single pass; Private AI targets the LLM-pipeline use case specifically, detecting 50+ entity types across 50+ languages with on-prem deployment for GDPR, HIPAA, CCPA compliance. Different category — they redact PII from text streams, they don't review designs. Not a direct overlap.

---

## What the skill catches that the tools don't

This is the value side of the comparison and stays robust:

- **Pure privacy failures** — Purpose Creep, Linkable Identifiers, Registration Vacuum, Precision Maximalism, Inference Without Consent, Shadow Profiling, Aggregation Creep, Re-identification via Join. None of these are Bearer rules (and most can't be expressed as one — they're design choices, not patterns in code).
- **Design-level God View** — Bearer catches sensitive data flowing to an internal endpoint; it can't reason about whether the endpoint *should exist* or whether access needs reason-given audit logging.
- **Consent UX** — Dark Patterns, Bundled Consent, Opt-out Default. These are HTML/UX issues, not data-flow issues.
- **Jurisdictional grounding** — Bearer cites CWE and OWASP A0X, not GDPR Art. 9 or BIPA.
- **Named-pattern vocabulary from privacypatterns.org** — Pseudonymous Identity, Sticky Policies, Layered Policy Design, Selective Disclosure, etc.
- **Real-world precedent** — Uber God View, BIPA settlements, FTC actions on health apps, Strava heatmap, AOL/Netflix re-identification.
- **IOPD process structure** — scoping, requirements, trade-off analysis, risk management, verification, monitoring.

---

## Real gaps in the skill, ordered by impact

These are things the tools surface that the skill currently doesn't.

### 1. Data Quality as a privacy risk (OWASP P7) — genuine miss

The skill treats data as either "collected or not" and "deleted or not" — it never asks whether stored data is *accurate and current*. But inaccurate data is a privacy risk: people get denied loans, flagged at borders, refused service, or sent unwanted communications because of stale or wrong records about them. GDPR Art. 5(1)(d) ("accuracy") makes this an explicit legal requirement. The skill should add a lens.

**Suggested addition (data lifecycle lens or its own section):**
> **Data accuracy and currency.** What data does the system act on, and how stale is it? Is there a path for users to correct it (Privacy Dashboard supports this, but the *acting* code also needs to honour corrections)? Are derived or cached records updated when the source changes? Are decisions made about users on data that may be years out of date? GDPR Art. 5(1)(d) requires accuracy; the FCRA imposes specific obligations on consumer-credit data accuracy.

### 2. Data Breach Response (OWASP P3) — under-weighted

The skill mentions the Data Breach Notification Pattern in one row of the patterns table. P3 makes it a top-10 risk. A privacy review of a real system should ask: is there a detection mechanism, a documented notification process, a regulator-contact runbook, a 72-hour timer (GDPR Art. 33), affected-user notification (Art. 34, CCPA §1798.82, US state notification laws, HBNR)?

**Suggested addition (new review lens or expansion of existing lifecycle lens):**
> **Breach detection and response.** What is the detection plan? Who is on-call? What is the documented notification path to the supervisory authority (GDPR 72-hour deadline) and to affected users? Is there a runbook? Are forensic artifacts preserved without expanding the harm (e.g. not copying breached data to more hands)? Has the team rehearsed?

### 3. PII in URL query strings — specific, common, missed

The skill says "PII in Logs" but URL query strings are a distinct surface: they end up in browser history, in `Referer` headers sent to third parties when the user navigates, in CDN access logs, in proxy logs, and in analytics. Bearer flags this explicitly because GET parameters can be cached and logged on multiple intermediaries even over HTTPS.

**Suggested addition (API design lens):**
> **PII in URL paths or query strings.** Email addresses, user IDs that double as identifiers, session tokens, or any sensitive value passed as a GET parameter rather than in a POST body or auth header. The URL ends up in browser history, `Referer` headers (leaking to any third-party script the next page loads), CDN access logs, intermediary proxy logs, and often in your own access logs. Use POST or move identifiers to headers; if a path parameter is unavoidable, use an opaque token rather than the value itself.

### 4. Exception/error messages as a distinct surface

The skill has "PII in Logs" and "Verbose Error Messages" but treats them as separate items, not as a coherent surface. Exception text and stack traces are a third leakage path (often into Sentry/Rollbar/Bugsnag, also flagged by Bearer). Worth a single combined item.

### 5. Named common third-party sinks — concrete reference

The skill's "Third-party Free-for-all" is abstract. Bearer's enumeration of specific vendors (Segment, Datadog, Sentry, Mixpanel-like analytics, ClickHouse, BigQuery, Google Analytics, Algolia) is a useful concrete checklist. Worth a small reference table — not in SKILL.md, but in a new `references/common-third-party-sinks.md` listing typical sinks by purpose (analytics, error tracking, observability, search, warehouse) with the data that commonly leaks into each.

### 6. Specific risky API surfaces beyond CORS

The skill mentions CORS. Adjacent issues Bearer enumerates and the skill doesn't:
- `Access-Control-Allow-Origin` reflecting an attacker-controlled value
- Trust-boundary violations (mixing trusted and untrusted data in one structure)
- Open redirect (used in phishing for session/credential theft)
- HTTP parameter pollution

These are security findings with privacy fallout. Small additions in the security anti-patterns table.

---

## What I'd actually do

Three changes, ordered by value:

1. **Add OWASP P7 (Data Quality) as a review lens.** Real gap, legally grounded (GDPR Art. 5(1)(d), FCRA), affects real users.
2. **Promote breach response from one row to a small lens.** P3 is too important to be one line.
3. **Add the URL-query-string item to the API design lens.** Concrete and common.

Skip:

- A `references/common-third-party-sinks.md` file would be marginally useful but the skill already covers the *concept*; enumerating vendors adds maintenance burden (the list ages fast) and only marginal lift.
- Detailed copies of Bearer's security rules — they're the security canon already covered in the OWASP A0X anti-patterns table; duplicating them would bloat without adding signal.

Want me to make those three additions to the skill?

* So I said 'yes'.
I then said:
*This tool is really a privacy analysis and repair tool rather than a design tool. Modify the name and invocation process to reflect that.

So Claude did, and responded:

Right — the name promised design but the tool ships analysis and repair. Let me rename and reshape the framing accordingly.Now update the H1 and intro paragraphs to match the analysis-and-repair framing:Update the output-format example header so generated reviews are titled correctly:Let me also do a final pass for any other places where the framing should match the new positioning:Three remaining "Privacy by Design" mentions, all intentional: (1) keyword in the description for users who still search by that term; (2) intro paragraph explicitly noting that PbD is the philosophy the tool analyzes against; (3) GDPR Article 25's official name ("data protection by design and by default") — not a tool-name reference. Good.

Now rename the folder, rebuild outputs and the zip:Done. Renamed and reframed.

**Changes:**

| | Before | After |
|---|---|---|
| Folder | `privacy-by-design/` | `privacy-analysis-and-repair/` |
| `name:` | `privacy-by-design` | `privacy-analysis-and-repair` |
| H1 | `# Privacy by Design Review` | `# Privacy Analysis and Repair` |
| Output H2 | `## Privacy by Design Review` | `## Privacy Analysis and Repair` |
| Role framing | "privacy-focused reviewer" | "privacy-focused analyst" |
| Description length | 884 chars | 944 chars (80 under limit) |

**Framing shifts in the intro:**

- Findings are now "paired with concrete repairs" rather than "tied to a concrete remediation." The deliverable is explicitly described as analysis-and-repair: every problem comes with a specific change to make.
- The relationship to Privacy by Design is made explicit and properly subordinated: PbD (Cavoukian's principles + IOPD Design Process Standard) is the *philosophy* the tool analyzes *against*; the tool itself does review and remediation on artefacts that already exist. No more name overpromise.

**Description-field reshaping:**

- Triggers now include "audit," "analysis," "fix," "remediation" alongside "review" — covers users who frame it as a fix-it task rather than a design task.
- "is designing any feature touching personal data" → "hands over code, schemas, or specs touching personal data" — repositioned from design-time to handover-of-existing-artefact.
- Kept "privacy by design" and "data minimisation" as trigger keywords because users still ask using those phrases even when they want analysis-and-repair.

**Three remaining "Privacy by Design" mentions in the file, all intentional:**

1. Description trigger keyword (so users searching "privacy by design review" still match).
2. Intro paragraph explicitly explaining that PbD is the underlying philosophy this tool analyzes against — that's the rename's whole point.
3. GDPR Article 25's official name ("data protection by design and by default") in the regulatory table — that's the law's actual title.

**Files for upload:**

- `privacy-analysis-and-repair.zip` (~22 KB) — upload-ready package
- `privacy-analysis-and-repair/SKILL.md` (487 lines) — the skill itself
- `privacy-analysis-and-repair/references/regulations-by-jurisdiction.md` (189 lines) — regional regulatory catalogue

Same install flow as before — drop the zip into Claude.ai's Skills upload (or the unzipped folder into `~/.claude/skills/` for Claude Code). The old `privacy-by-design.zip` and its folder have been removed from outputs to avoid confusion. If you already installed the previous version, uninstall it first so you don't end up with two skills competing on overlapping triggers.


