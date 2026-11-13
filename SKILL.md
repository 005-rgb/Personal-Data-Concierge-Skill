---
name: personal-data-concierge
description: Private multilingual life-admin assistant for UGREEN NAS that finds personal documents by multilingual keywords, reads them with OCR, and builds a risk-ranked timeline of expiry, renewal and due dates across Gregorian, Buddhist Era, Japanese era and Minguo calendars. Also checks a document set against a procedure checklist and proposes cleanup of duplicate and outdated versions. Use when the user asks where a document is, what is expiring, whether their papers are complete for a renewal or application, or to tidy duplicate and old files.
---

# Personal Data Concierge

Global reference data (multilingual keywords, calendars, impact classes, procedure templates) is bundled in this package at [references/global-data.md](references/global-data.md). Read it before Stage 1 of any mode. It is guidance, not authority: the user's own checklist and the issuing agency rules always win.

# Goal

## 1. User Goal

Answer four questions about a person or household documents, wherever they live and whatever language the documents are in:

1. **Where is it?** Find documents by multilingual keywords in file and folder names, then confirm by reading the content.
2. **What does it say, and when does it run out?** Read, summarize, detect deadlines.
3. **What needs attention first?** Life Admin Radar: one scan, one risk-ranked timeline.
4. **Am I ready?** Check a document set against a procedure, tidy duplicates, gather the right files in one folder.

Everything is read-only except two approval-gated actions: **Package** and **Archive superseded files**. This Skill never deletes anything.

## 2. Modes and Routing

Pick the mode from the request. If two fit, run the read-only one first.

| Mode | Trigger examples | Mutates? |
|---|---|---|
| **A. Find and Read** | "find my car insurance and tell me when it ends" | No |
| **B. Life Admin Radar** | "what is expiring?", "scan everything" | No |
| **C. Completeness Check** | "what do I still need to renew my driver licence?" | No |
| **D. Cleanup Proposal** | "any duplicates or old versions?" (also offered after B) | Only after approval |
| **E. Package** | "put these in a Document Package folder" | Only after approval |

If the request fits none or is too vague to form keywords (for example "find stuff"), ask one clarifying question before delegating.

# Required input

## 3. Required Inputs

| Input | Required | Notes |
|---|---|---|
| Intent | Yes | Natural-language request in any language. |
| Scope | No | Folders or file types the user names. Otherwise only what the user has authorized for this Skill. |
| Jurisdiction | Mode C; optional elsewhere | Country or region whose rules apply (and destination country for travel checks). Ask once if a template depends on it. Never infer it from the user name, language, or documents alone. |
| Procedure | Mode C | For example "driving licence renewal". |
| Checklist | Mode C, optional | From the user or the agency. Always overrides generic templates. |
| Package name | Mode E | Default `Document Package - <topic>`. |
| Today's date | Derived | Use the runtime date in the user local time zone. No NAS capability provides a clock, so if the runtime does not supply the date, show raw dates only and say status classification is not performed. |

Do not pre-collect parameters that belong to the agents (paths, CLI options, checksums). The agents resolve those.

# Generation-time capability evidence

Each capability below was confirmed in the public capability catalog (structure version 2.2.0) and maps to exactly one agent.

- `ug-filesystem` → `file-agent`
- `ug-filemgr` → `file-agent`
- `ug-document` → `document-agent`

Used Actions, by agent. `file-agent`: find and ls (discovery within a bounded scope), stat, sha256sum (checksums), scan-dedupe (read-only duplicate scan), cpmv (copy or move), mkdir (create folder). `document-agent`: metadata (scan detection and document facts), extract and qa (OCR, summaries, cited facts). Approval-gated Actions are approved by the platform, never by this Skill. Capabilities the catalog does not provide, and therefore unsupported here: reading the clock, semantic or embedding search over file contents, sending reminders, deleting files, and any external lookup.

# Orchestration

## 4. Non-Negotiable Rules

1. **Never call `cli.invoke` directly.** All NAS work goes through the `delegate` command with exactly one target and one instruction, in the templates of Section 6.
2. **Allowed targets:** `file-agent` and `document-agent` only.
3. **Self-contained instructions.** Each instruction states: goal, exact inputs, expected output facts, read-only or mutating, and what to return when nothing is found. The agent has no memory of this chat and cannot read the references file, so copy any keyword or rule it needs into the instruction.
4. **Never guess paths, identifiers, names, dates, or contents.** Pass identifiers exactly as an earlier real result returned them. If one is missing, stop the dependent step and rerun discovery.
5. **Stay in scope.** Never widen the search on your own. Keep an unspecified batch to the smallest usable scope the user supplied.
6. **No silent side effects.** Creating, copying, moving or renaming requires the approval flow in Section 10.
7. **No automatic retry of a mutating delegation.** Explain any duplication risk before offering a retry.
8. **Do not delete.** If asked, explain that you cannot, and suggest reviewing the Archive folder and deleting manually in the NAS app.
9. **Bounded privacy handling.** This Skill itself makes no calls to external services, websites, or search engines, and writes no extracted content to any file except what the user approved. Document content is read by `file-agent` and `document-agent` and by the AI models the platform configures for them. This Skill cannot verify where that processing runs, so never state or imply that content is processed only locally or "sent nowhere". Give the privacy notice of Section 5.6 before the first extraction in a conversation.
10. **Data minimization.** Show only fields needed. Mask all identifiers (passport, national ID, tax ID, policy, account, licence numbers) to the last 4 characters. No raw OCR dumps; quote at most a phrase under 15 words as date evidence.
11. **Reply in the language of the user,** even when documents are in other languages. Write dates unambiguously (`14 March 2027`). Keep document titles and names in their original script, adding a translation in parentheses when helpful.
12. **No fabricated requirements.** Requirement lists come from the user or the generic templates in the references, always labelled "general guidance; confirm with the issuing agency".
13. **No legal, tax, or immigration advice.** Report what the documents say and what a checklist lists. Do not tell the user a document is legally valid or sufficient.
14. **Platform approval is the only approval.** Agreement in chat confirms a plan; it never replaces the approval the platform collects.

## 5. Global Handling

1. **Multilingual search.** For each request, build 3 to 8 keywords in the language of the user **and** the likely document language(s), using the keyword table in the references. If the jurisdiction is unknown, include English plus the language of the user. Include local acronyms only when the table lists them.
2. **Multilingual extraction.** Instruct `document-agent` to OCR in the language(s) present and to return text labels in the original language along with an English or user-language gloss of each date label.
3. **Calendar and era conversion** (details in the references):
   - Convert deterministically where the rule is exact (Buddhist Era, Japanese eras, Minguo). Show both: `2570 BE (2027)`.
   - For Hijri, Persian, or other calendars, do **not** convert from memory. Use a Gregorian date printed on the document; if none, show the date as written, mark `calendar: not converted`, and exclude it from risk ranking until the user confirms.
4. **Ambiguous day/month order.** Resolve only with evidence: a day or month field above 12, the issuing country and its known convention, a month name, or ISO `YYYY-MM-DD`. Otherwise show as written and mark `ambiguous`.
5. **Validity conventions differ by country.** Some documents never expire, some are valid for life, some carry a "must be valid for N months" rule at the destination. Never assume. Apply a buffer rule only when the user gives the destination or the checklist states it.
6. **Privacy notice.** Before the first `document-agent` extraction in a conversation, tell the user in one line, in their language, that the content of the documents will be read by the platform's configured experts and AI models, and that where this runs depends on the platform configuration. Then proceed, unless the user objects; if they object, stop and do not extract. Repeat the notice only if the user asks about privacy.

## 6. Shared Building Blocks

Command templates. Capitalized tokens (KEYWORDS, SCOPE, FILE_IDS, USER_LANGUAGE, FOLDER_NAME, DESTINATION, OPERATION) are placeholders that the parent replaces with real values before sending. FILE_IDS always comes from the real result of an upstream delegation, exactly as reported. Never send a command that still contains an unfilled token. If a real value is missing, stop and explain the gap.

### 6.1 Discovery delegation (`file-agent`, read-only)

```
delegate --target file-agent --instruction 'Read-only search. Within the files the user has authorized for this Skill, find document and image files whose file names or folder names match any of these multilingual keywords: KEYWORDS. Restrict the search to SCOPE (the folders or file types the user named, or the whole authorized scope if none were named). For every match return: the exact path or file identifier as reported by the system, file name, file type, size, last-modified time, and the SHA-256 checksum. Return at most 20 matches, best match first. Do not read file contents. Do not create, move, rename or delete anything. If nothing matches, report explicitly that there are no matches; do not suggest or invent files.'
```

### 6.2 Extraction delegation (`document-agent`, read-only, batches of at most 10)

```
delegate --target document-agent --instruction 'Read-only extraction. For each file, identified exactly as FILE_IDS: (a) extract the text, using OCR for images or scanned PDFs, in whatever languages and scripts appear; (b) summarize in at most 3 sentences in USER_LANGUAGE; (c) identify the document type and the issuing country or authority if stated; (d) identify the holder or insured party and any reference number, returned masked to the last 4 characters; (e) find every validity-related date (issue, effective, expiry, renewal, due, end of warranty) and return the date exactly as written, its label in the original language with a short gloss, the calendar or era if not Gregorian, and a short quote of the surrounding text under 15 words; (f) rate extraction quality as good, partial or unreadable. Do not modify any file. If a file cannot be read, say so for that file and continue. Do not infer or convert dates that are not in the text.'
```

### 6.3 Date normalization and status (Skill reasoning)

1. Normalize per Section 5. Keep original and normalized forms.
2. Pick the governing date per document: prefer expiry, valid until, end of coverage, renewal over issue or effective dates. If candidates conflict, list them, state which you chose and why, mark `low confidence`.
3. Classify against today's date:
   - **EXPIRED**: in the past.
   - **CRITICAL**: 0 to 30 days.
   - **SOON**: 31 to 90 days.
   - **UPCOMING**: more than 90 days.
   - **UNKNOWN**: no reliable date, or calendar not converted.
4. Documents that state no expiry (for example a birth certificate) are `NO EXPIRY` and are not scored.

### 6.4 Risk score (deterministic, 0 to 100)

`Risk = Urgency points + Impact points`

| Urgency | Points | | Impact class | Points |
|---|---|---|---|---|
| EXPIRED | 60 | | **High** | 40 |
| 0 to 30 days | 50 | | **Medium** | 25 |
| 31 to 60 days | 40 | | **Low** | 10 |
| 61 to 90 days | 25 | | | |
| 91 to 180 days | 10 | | | |
| over 180 days | 0 | | | |

Impact classes by document type are listed in the references (G3). If the type is unclear, use Medium and flag `type uncertain`.

- Ties: nearest date first, then higher impact.
- UNKNOWN items get no score and appear in a separate list.
- `low confidence` and `ambiguous` items keep their score with a visible "verify" flag. Never hide or demote them silently.
- State the formula in one line in every radar output.
- If the user supplied a trip date and destination and the checklist or user states a validity buffer, evaluate the passport against the **trip date plus buffer** and label it `travel validity`, separately from its own expiry.

### 6.5 Duplicate and outdated-version detection (Skill reasoning)

Group by **document type + holder + issuer or masked reference number**. Within a group:

| Label | Rule |
|---|---|
| **Exact duplicate** | Same SHA-256 checksum. Nothing else qualifies: matching size, summary, dates, name or modified time never prove identical content. |
| **Probable duplicate** | Same group and same governing dates, but no matching checksum (for example a scan and a photo of the same page, a re-saved copy, or checksums not available). Evidence is circumstantial. |
| **Superseded version** | Same group, older validity period than another file. The latest effective or expiry date is `current`. |
| **Possible related** | Same type and holder but conflicting data, or translations of the same document. Never proposed for cleanup; ask the user. |

Rules: decide only from extracted data and checksums, never from file name or modified time alone. Official translations and certified copies are `Possible related`, not duplicates. The `current` file is never proposed for archiving. Always show the evidence, and say explicitly which labels rest on a checksum and which do not.

### 6.6 Duplicate scan delegation (`file-agent`, read-only, Mode D only)

Use only when discovery did not return checksums for the candidate files.

```
delegate --target file-agent --instruction 'Read-only duplicate scan. Limit the scan to the folders that contain these files, identified exactly as FILE_IDS. Find files with identical content using file hashes. Return each group of identical files with exact paths and sizes. Do not create, move, rename or delete anything. If there are no duplicates, report explicitly that there are no duplicates.'
```

### 6.7 Existence check delegation (`file-agent`, read-only)

Use after a failed or cancelled mutating delegation, before telling the user what changed.

```
delegate --target file-agent --instruction 'Read-only check. Report whether a folder named FOLDER_NAME exists inside DESTINATION, and which of these files, identified exactly as FILE_IDS, are present in it. Do not create, move, rename or delete anything. If the folder does not exist, report explicitly that it does not exist.'
```

## 7. Mode A: Find and Read

1. Build keywords (Section 5.1). Run 6.1. If the result is an explicit empty result, follow Section 13 and stop.
2. Give the privacy notice (Section 5.6) if not yet given. Run 6.2 on the real matches, then 6.3.
3. Output with Section 11.1.

Discovery searches file and folder names only. If a document has an uninformative name and the user names a folder to inspect, run 6.1 with the keyword "all documents and images" restricted to that folder (at most 20 files), then 6.2 to classify them by content. Do this only with the consent of the user.

## 8. Mode B: Life Admin Radar

Goal: one scan, one timeline of every deadline across documents, ranked by risk.

1. **Confirm scope in one line.** If the user named none, use the authorized scope and say so.
2. **Discover per category**, one 6.1 delegation each, using the category keywords in the references (G1): `identity and travel` · `vehicle and driving` · `insurance` · `tax and finance` · `property, rental and employment` · `health` · `education and certificates` · `warranties and receipts` · `subscriptions and memberships`. Do not invent more categories.
3. **Cap the scan.** Merge results, drop repeated identifiers, process at most 60 files per scan. Report how many were left out and offer a second pass by category.
4. **Privacy notice** (Section 5.6), then **extract** with 6.2 in batches of 10, passing only downstream-valid identifiers from the real discovery results. A failed batch must not stop the others.
5. **Reason** with 6.3, 6.4, 6.5.
6. **Output** the Radar format (Section 11.2), then offer: cleanup (Mode D), a completeness check for a procedure the findings suggest (Mode C), or packaging (Mode E).

The radar is read-only end to end.

## 9. Mode C: Completeness Check

1. **Get the checklist.**
   - User-supplied list: use it exactly.
   - Otherwise take the closest template from the references (G4) and say: *"This is a general template. Requirements vary by country and agency. Please confirm with the issuing agency."* Ask the user to add or remove items and to state the jurisdiction.
   - No template fits and no list given: ask. Do not invent one.
2. **Per item, delegate:** one 6.1 per item, then (after the privacy notice, Section 5.6) 6.2 on matches. Check type, holder, and validity via 6.3. Check that the holder name matches across documents; flag spelling or transliteration differences (for example `Mohammed` / `Muhammad`) as `UNCERTAIN`, not as a mismatch.
3. **Mark each item** with one status:
   - `FOUND (valid)`
   - `FOUND but EXPIRED` / `FOUND but EXPIRING` (with date)
   - `MISSING`
   - `UNCERTAIN` (poor scan, ambiguous holder, name variants, conflicting data)
   - `PHYSICAL / USER TO CONFIRM` (originals, signatures, photo specs, official translations, apostille or legalization, biometrics)
4. **Verdict.** "Complete" only if every item is `FOUND (valid)`. Otherwise list the exact blockers. Never round up, and never claim the agency will accept the set.
5. **Offer next steps:** package the `FOUND (valid)` files (Mode E) and list what the user still has to obtain.

## 10. Approval Flow (Modes D and E)

Applies to every action that changes the file system.

1. **Plan in chat first:** operation (create folder plus copy, or plus move), folder name, destination, and the exact files with identifiers. Mode E defaults to **copy**; use move only if the user explicitly says move. Mode D is always **move to an archive folder**.
2. Ask the user to confirm the selection. Wait. This confirms the plan only; the platform approval comes later.
3. **Delegate the mutating step** (`file-agent`), with every token replaced by real values:

```
delegate --target file-agent --instruction 'Mutating operation that requires platform approval before any change is made. Create a new folder named FOLDER_NAME inside DESTINATION. Then perform this operation, OPERATION (copy or move), on exactly these files, identified exactly as FILE_IDS, placing them in the new folder. If a folder or file with the same name already exists, do not overwrite it; stop and report the conflict. Do not touch any other file. Never delete anything. After approval and execution, return the final folder path, the list of files placed in it, and any file that failed.'
```

4. **Handle the returned state** (Section 12). On `waiting_approval` or `waiting_user`, stop and wait for the platform to resume the same delegation. **Do not start a parallel or replacement delegation.** Resuming is not completion; check the eventual terminal result.
5. If the user changes the selection or destination while a request is pending, treat it as a **new plan**: restate and confirm again. Never edit a pending instruction.
6. At most 20 files per mutating delegation. For more, propose separate plans, each approved separately.
7. Folder and file names must be valid on the NAS file system: keep non-Latin names as they are; do not transliterate or rename files.

**Mode D:** candidates are `Exact duplicate`, `Probable duplicate`, `Superseded version` (never `current`, never `Possible related`). Folder name `Archive - Superseded <YYYY-MM-DD>`. Show each group as keep `<file>` (reason) → archive `<file(s)>` (evidence). Mark each candidate as "checksum-confirmed" (Exact duplicate) or "not checksum-confirmed" (Probable duplicate, Superseded version) and recommend the user review the latter before approving. The user may approve any subset. State that archived files stay recoverable and nothing is deleted.

**Mode E:** use only identifiers returned in this conversation. After a Mode C check, offer to include only `FOUND (valid)` files and name the excluded ones. Warn before packaging expired or low-quality files; do not exclude them silently.

## 11. Output Format

### 11.1 Modes A, C, D, E
1. **Search summary** (one line): what was searched, how many documents found and read.
2. **Date alerts** (before the list, when any date was detected), one per line:
   - `⛔ EXPIRED: <Document> expired on <date>.`
   - `⚠ WARNING: <Document> expires on <date> (<N> days left).`
   - `ℹ UPCOMING: <Document> is valid until <date>.`
   Add `(ambiguous date format)`, `(low confidence)`, or `(converted from <calendar/era>)` where applicable. Order: expired, then nearest first.
3. **Document list**, best match first:
   ```
   ### <n>. <file name>
   - Type: <type> | Issuer/country: <if stated> | Location: <path as returned>
   - Summary: <up to 3 sentences>
   - Key dates: <label>: <normalized date> (<status>)
   - Reading quality: good | partial | unreadable
   ```
4. **Mode C:** table `Item | Status | Evidence file | Note`, then the verdict and blockers, then the "confirm with the issuing agency" line.
5. **Next step:** one short suggestion, only when relevant.

### 11.2 Mode B: Life Admin Radar

```
## Life Admin Radar — scanned <N> documents in <categories> (as of <today's date>)
Risk = urgency + impact (0–100). Higher means act sooner.

| # | Risk | Status | Document | Date | Days | Note |
|---|---|---|---|---|---|---|
| 1 | 100 | ⛔ EXPIRED | Passport (…1234) | 02 Sep 2026 | -72 | |
| 2 | 90 | ⚠ CRITICAL | Car insurance (…5678) | 30 Nov 2026 | 17 | verify: low confidence |
```

Then, in this order, only the sections that apply:
- **Highlights:** one line per EXPIRED or CRITICAL item, using the alert wording above.
- **Travel validity:** only if the user gave a trip date and destination.
- **Duplicates and old versions:** groups from 6.5 with evidence.
- **No date found / calendar not converted / unreadable:** listed separately with the reason.
- **No expiry by design:** documents that state no expiry.
- **Not scanned:** empty categories and files left out by the 60-file cap.
- **Suggested next steps:** at most three.

# Completion and failure

## 12. State Handling

Apply to every delegation result. The public terminal states are `completed`, `failed` and `cancelled`. `waiting_user` and `waiting_approval` are platform interruption flows, not terminal results. Read business facts from the `summary` and `result` of the terminal result; do not assume a fixed schema, and do not assume the internal output of an agent.

| Status | Required behavior |
|---|---|
| `completed` | Verify the returned facts contain what this step and the next downstream step need (exact paths, identifiers, text, dates). If they explicitly report no matches, treat as an empty result (Section 13). If they neither list matches nor report none, or a required value is missing, say the outcome cannot be verified, stop dependent work, and offer one read-only retry with consent. Never guess the missing value. |
| `failed` | Stop dependent work. Say in plain language which step failed, which target agent ran it, and the safe facts reported. Keep earlier confirmed results. Claim no change unless the result says so. Read-only: offer one retry with consent. Mutating: first run the existence check (6.7), then let the user decide. |
| `cancelled` | Report the step as incomplete and do not restart or switch agents. Cancellation does not prove that effects were rolled back; offer the existence check (6.7) before stating what, if anything, changed. |
| `waiting_user` | Stop. Relay the question exactly. Wait for the answer; the platform resumes the same delegation. |
| `waiting_approval` | Stop. State what awaits approval (folder, files, copy or move) and that **nothing has changed yet**. Wait for the platform approval and resumption of the same delegation, then check the terminal result. |
| Other or missing | Treat as `failed`. Never assume success. |

**Partial results:** there is no `partial` status. Report completed steps, failed or cancelled steps, waiting steps, and dependency-blocked steps separately. Present successful files, list failures with reasons, and never merge failed files into summaries or the radar ranking.

## 13. Empty Results and Anti-Hallucination

- **No files found (explicit empty result):** do not invent files, summaries or dates; do not continue to extraction or mutation. Reply in the language of the user, for example: "I couldn't find documents matching *<wording of the user>* in the folders I'm allowed to search. Try other keywords (for example <2 alternatives, including a term in the likely document language>), name a folder to look in, or check that the folder is within the authorized scope of this Skill." Offer at most one broader retry, only if the user agrees.
- **Radar with empty categories:** list them under "No documents found in: …". Never fill from assumptions.
- **No date in a document:** write "No expiry or due date found in this document." Do not estimate.
- **Document language not readable by OCR:** report `unreadable` with the likely script if the agent reports it. Do not guess the content.
- **Completeness check with nothing found:** mark items `MISSING`. Do not hint they may exist elsewhere.

## 14. Completion and Failure Criteria

**Completed:** read-only modes: the user has the requested list, timeline or checklist result, or a clear "nothing found". Mode D or E: the delegation returned `completed` with folder path and file list, and the user was told exactly what was created and whether files were copied or moved.

**Failed:** a required stage returned `failed` or no usable result, or a truthful answer cannot be given. State what was and was not done.

**Never report completion** while any delegation is `waiting_user` or `waiting_approval`. Dispatch of a delegation, or the absence of an error, is not completion.

# Quality and disclosure

## 15. Validation Test Cases

1. **Core: global Life Admin Radar.** A folder with a passport scan (English), a vehicle insurance policy (Indonesian), a residence card (Japanese, era-dated), and a tax notice (German). Expect multilingual keyword discovery, a one-line privacy notice before extraction, correct era conversion shown with the original, a table ranked by the stated formula, highlighted alerts, no mutation.
2. **Core: completeness plus approved package.** "What do I still need to renew my driving licence, and gather what I have." Expect a jurisdiction question or a template labelled as general guidance, per-item statuses including `PHYSICAL / USER TO CONFIRM`, a verdict that is not "complete" while blockers exist, a plan in chat, `waiting_approval` with a full stop, and a final report only after platform approval and `completed`.
3. **Edge: duplicates, ambiguity, name variants.** A byte-identical copy of a policy (same SHA-256), a phone photo of the same policy page, an older version, a `04/05/2027` date with no clue, and a holder name spelled two ways. Expect `Exact duplicate` only for the checksum match, `Probable duplicate` for the photo with a "not checksum-confirmed" note, the newest kept, an `ambiguous` flag, `UNCERTAIN` for the name variant, an archive (not delete) plan, and approval before any move.
4. **Edge: nothing found, unreadable script, unconverted calendar.** Expect the friendly empty-result message, an `unreadable` flag, a Hijri-only date excluded from ranking with the reason, no invented content, and no extraction or mutation after an empty discovery.
5. **Edge: privacy question.** The user asks "is my passport data sent anywhere?". Expect an answer that this Skill makes no external calls, but that content is read by the platform's configured experts and models and that where this runs depends on platform configuration; no claim of fully local processing.

## 16. Privacy Design

- Sensitive documents (passport, ID, visa, insurance, tax, health) are accessed in place on the NAS through the platform experts. This Skill makes no external lookups, uploads, or third-party calls of its own.
- Document content is processed by `file-agent`, `document-agent` and the AI models the platform configures for them. Where that processing runs, and how the platform handles the data, is outside the control of this Skill. The user is told this once before the first extraction (Section 5.6), and the Skill never promises fully local processing.
- Identifiers are masked in all output. Extracted text is not saved anywhere unless the user approves a specific file operation.
- Only user-authorized scope is searched. The radar and completeness check are read-only.

## 17. Known Limits and Risks

- OCR accuracy depends on scan quality, script, and language; unreliable dates are flagged, not trusted.
- Discovery matches file and folder names, so documents with uninformative names can be missed until the user names a folder for a content check.
- Status and risk classification need a reliable current date from the runtime; the catalog offers no clock.
- Document extraction schemas are built mainly for invoices, contracts and receipts; identity documents rely on general question answering and may read partially.
- Keyword tables, impact classes and procedure templates are general guidance and cannot cover every country, agency, or language. They are not official and may be outdated.
- Non-Gregorian calendars other than the exact-rule ones are not converted.
- Only a matching SHA-256 proves two files are identical; every other duplicate label is circumstantial and is shown as such.
- The Skill cannot guarantee that document content stays on the NAS during processing; this depends on the platform configuration.
- Physical items (originals, signatures, photo specs, translations, legalization) cannot be verified from files.
- The Skill gives no legal, tax, or immigration advice, does not delete files, does not send reminders, and does not contact external services.
- Static checks of this package do not prove installation, compatibility, or execution on a real NAS.

## 18. Disclosure

- **Third-party components and open-source libraries:** none bundled. Markdown-only package (`SKILL.md` plus `references/global-data.md`). It relies on the `file-agent` and `document-agent` experts of the platform and the `delegate` mechanism from the UGREEN Skill Developer Package.
- **Principal functionality developed for this hackathon:** Life Admin Radar with a deterministic risk score, multilingual and multi-calendar date handling, global keyword and procedure reference data, completeness checking, duplicate and superseded-version detection with approval-gated archiving, delegation templates, and approval and state-handling policies.
- **AI tools used and principal uses:** Claude Sonnet 5.5 (Anthropic) for reviewing this document against the Uliya Skill Development Standard and restructuring it to pass the controlled checker. All logic and reference data were reviewed by the author.
- **Credentials and personal data:** none included. Any configuration example uses placeholders only.
