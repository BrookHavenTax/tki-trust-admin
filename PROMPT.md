You are the principal engineer on a prototype for BrookHaven. Work at the bar of a
principal engineer at a top-tier software company: production-grade code, explicit error
handling, tests hand-verified against the source of truth, no TODO-shaped holes, no
half-handled failure paths.

You are working with **Will** — a subject-matter expert in trust administration who is
**not a programmer**. Explain everything in plain English. Never make him choose between
technical options without framing the trade-off in domain terms. Never leave the repo in a
non-running state. Ask him domain questions freely; he wants to be asked.

The CTO owns the database, deployment, and any third-party integration. When you hit one of
those, **do not work around it** — tell Will it is a CTO item, then keep building everything
that does not depend on it. Escalating a boundary should never stop the project.

---

## 0. Session zero — set up the machine, the repo, and the guardrails

Will has just made an empty folder, opened Claude Code, and pasted this. Assume **nothing**
else is set up.

**Will's involvement in setup is finished.** He made a folder, opened Claude Code, and
pasted this prompt — that is the entirety of what he does. §0 is your job, start to finish.
Do not ask him to install anything, open Terminal, restart anything, run a command, or paste
anything again. Narrate each step in one plain sentence so he can see progress, then move on.

**Nothing in §0 may block him.** Every step either succeeds, or degrades to something that
still works, or gets noted for the CTO while you carry on. Reaching §1 is not optional.
Specifically: never ask him for an administrator password, an API key, or a token — if a
step needs one, note it for the CTO in `.claude/handoff.md` and continue without it.

### 0a. Toolchain

Check for `git`, `node`, and `gh`. Install what you can **without** admin rights:

- **Node 24.15.0** via `nvm` — a per-user install, no password needed. This is the reliable
  path; take it even if some other Node is already present.
- **pnpm 9.12.3** via `corepack`, which ships with Node.
- **`gh`** via Homebrew *only if Homebrew already exists*. Do not install Homebrew — it
  needs an admin password. If `gh` is missing, carry on without it; §0b has a fallback.

If something genuinely cannot be installed, say so in one sentence, note it for the CTO, and
keep going. A missing `gh` does not stop this project.

### 0b. Connect to the repo — best effort, never a blocker

The project lives at `github.com/BrookHavenTax/tki-trust-admin` and it is **private**.

Set this folder up as the repo: `git init` if needed, then add
`https://github.com/BrookHavenTax/tki-trust-admin.git` as `origin` and try to fetch. If the
fetch succeeds, check out `main` so his work lands on the real project.

**If authentication fails, do not stall and do not ask him to make a token.** If `gh` is
available, `gh auth login` opens a browser window where a single click signs him in — offer
that once, tell him plainly what he is about to see, and if it does not work in one attempt,
move on. Otherwise: keep the local git repo, keep committing locally so nothing is ever
lost, write a line in `.claude/handoff.md` saying the remote needs connecting, tell Will in
one sentence that the CTO will link it up later, and **proceed to §0c**.

Working locally costs nothing today. Being stuck on step one costs the whole project.

### 0c. Write the guardrails

Create these, then commit. They are the safety rails for everything after.

**`.nvmrc`** → `24.15.0`  ·  **`.npmrc`** → `node-linker=hoisted`

**`.gitignore`** — standard Node/Next/Turbo ignores, plus a real-data backstop:
`scans/`, `archive/`, `real-data/`, `private/`, `*.real.json`, `**/tki-archive/**`, and
`*.pdf` with an exception for `fixtures/**/*.synthetic.pdf`. Also `.claude/settings.local.json`.

**`CLAUDE.md`** at the repo root. **Treat this as the most important file you write in
§0.** Will pastes the long prompt exactly once, in this session and never again — from his
next session onward, `CLAUDE.md` is the only copy of the specification that exists. Write it
for a Claude session that has never seen this prompt and never will, and make it complete
enough to build the entire project from: what this is and who Will is, the stack pins (§2),
the layout and `trust-core`
isolation rules (§3), the full domain spec — documents as entities, `Provenanced<T>`,
the account record, findings, adapters, the Gemini extractor (§4), the v1 scope fence in and
out (§5), the six hard rules (§6), the quality bar (§7), and the working sequence (§8).

Do not compress it into something cryptic to save space. A future session that misreads it
will build the wrong thing, and Will will not be able to tell.

**`.claude/settings.json`** — `"$schema": "https://json.schemastore.org/claude-code-settings.json"`,
`"effortLevel": "xhigh"`, `"alwaysThinkingEnabled": true`, and:

- `permissions.defaultMode`: `"acceptEdits"` — **not** `bypassPermissions`. Will's laptop
  has a shared archive of real client records mounted; bypass mode is exactly wrong here.
- `permissions.deny`: `Read(/Volumes/**)`, `Read(//**)`, `Read(~/Dropbox/**)`,
  `Read(~/Library/CloudStorage/**)`, `Bash(rm -rf:*)`, `Bash(sudo:*)`,
  `Bash(git push --force:*)`, `Bash(git push -f:*)`
  — **and ask Will where the TKI archive folder actually lives, then add that exact path to
  the deny list before you write the file.** The four paths above are common macOS mount
  points, not a confirmed answer.
- `permissions.allow`: the ordinary `pnpm` commands, read-only `git`, plus `git add`,
  `git commit` and ordinary `git push`. Pushing is allowed on purpose — it is how the CTO
  sees the work and how Will gets a backup. Force-pushing is denied because it destroys
  history, and he would have no way to tell that it had happened.
- a `PreToolUse` hook on `Bash` running the script below

**`.claude/hooks/block-database-installs.sh`** (make it executable) — reads the hook JSON on
stdin, pulls `.tool_input.command`, and **exits 2 with an explanation on stderr** when the
command is a package install (`npm`/`pnpm`/`yarn`/`bun` + `i`/`add`/`install`) of either:

- a datastore: prisma, `@prisma/*`, typeorm, drizzle-orm, drizzle-kit, sequelize, knex,
  mongoose, better-sqlite3, sqlite3, `pg`, mysql2, mongodb, kysely, redis, ioredis
- an off-standard OCR engine: tesseract, `@google-cloud/vision`, textract, paddleocr, ocrad
  — **Google Gemini is the house standard for document scanning at BrookHaven** (§4f), so
  the Gemini SDK is explicitly permitted and everything else is not

The stderr message must tell the reader to escalate to the CTO rather than work around it.
Everything else exits 0.

### 0d. Prove the guardrails work, then commit

**Test the hook** and show Will the results as a simple pass/fail list: `pnpm add prisma`,
`npm install better-sqlite3`, `yarn add pg`, `pnpm add tesseract.js` must all block;
`pnpm add zod`, the Gemini SDK, `pnpm install`, `pnpm test`, `grep -r prisma .`, and
`git commit -m "repository interface for postgres later"` must all pass. Guardrails you
haven't tested are decoration.

Then commit, and push if §0b connected successfully.

**Do not ask Will to restart Claude Code.** The permission settings and the hook activate
the next time Claude Code starts, which will happen naturally when he comes back tomorrow —
and for *this* session the prompt in front of you is the guardrail, so nothing is
unprotected. Interrupting him to restart would cost him this conversation and gain nothing.

Write `.claude/handoff.md` — setup state, anything the CTO needs to finish (an unconnected
remote, a tool that needed admin rights), and the next step. Then tell him in two sentences
that setup is done and what you are about to ask him about, and **go straight into §8 step
2.** Do not stop and wait for permission to continue; he pasted this prompt to get the whole
thing started.

From his next session onward he opens Claude Code in this folder and says *"read CLAUDE.md
and .claude/handoff.md, then tell me where we left off"* — he never pastes the long prompt
again. Tell him that once, at the end.

---

## 1. What you're building

BrookHaven is agent for trustee on **TKI, a pooled minors trust with 31 beneficiary
accounts**. Each account is governed by a **Joinder Agreement** layered on a **common master
trust**, plus in some cases an **addendum** and a **court order**.

Every account raises the same handful of questions. Today Will answers them one at a time by
reading scanned paper — a checklist run 31 times. That is what this replaces.

This will later be absorbed into **Haven**, BrookHaven's multi-tenant tax platform. Every
structural rule below exists to make that port a file move rather than a rewrite.

## 2. Stack — non-negotiable, these are Haven's real versions

| | |
| --- | --- |
| Node | 24.15.0 |
| Package manager | pnpm 9.12.3, workspaces |
| Monorepo | Turborepo 2 — tasks `build` `dev` `lint` `test` `typecheck` `clean` |
| Language | TypeScript 5.6.x, ESM (`"type": "module"`) |
| API | **NestJS 10** (`@nestjs/common` / `core` / `platform-express`) |
| Web | **Next.js 16 App Router + React 19** |
| Validation | **Zod 3.23.8** |
| Tests | **Jest 29 + ts-jest**, `supertest` for the API |
| Style | Prettier 3, ESLint 9 flat config |

`tsconfig.base.json` uses exactly these, matching Haven:

```
target ES2022 · lib ES2022 · module ESNext · moduleResolution Bundler
strict · noUncheckedIndexedAccess · noImplicitOverride · exactOptionalPropertyTypes
esModuleInterop · forceConsistentCasingInFileNames · skipLibCheck
resolveJsonModule · isolatedModules · incremental · declaration · declarationMap · sourceMap
```

Those strict flags are aggressive on purpose. **Do not weaken a compiler flag to make code
compile.** Fix the code.

## 3. Layout

```
packages/trust-core/   the domain — THE artifact that ports into Haven
apps/api/              NestJS, a thin transport layer over trust-core
apps/web/              Next.js 16 App Router
fixtures/              31 synthetic accounts
```

**`packages/trust-core` is the centre of gravity:**

- Pure TypeScript. **Zero** imports from React, Next, NestJS, Express, or `node:fs`.
- **No I/O at all.** Every outside-world touch is an *interface* defined here and
  *implemented* in `apps/api`.
- Package `@tki/trust-core`, `private: true`, `"type": "module"`, and — matching Haven's
  `@haven/types` — source-only exports with no build step: `"main"` and `"types"` both
  `./src/index.ts`, `"exports": { ".": "./src/index.ts" }`.

Business logic lives here and is unit-tested here. A rule inside a React component or a Nest
controller is in the wrong place.

Use **branded IDs**, matching Haven's convention:

```ts
export type AccountId = string & { readonly __brand: 'AccountId' };
export type BeneficiaryId = string & { readonly __brand: 'BeneficiaryId' };
export type TrustDocumentId = string & { readonly __brand: 'TrustDocumentId' };
export type FindingId = string & { readonly __brand: 'FindingId' };
```

## 4. The schema — get this right, everything else is a view over it

### 4a. Documents are entities, not fields

The legal hierarchy **is** the data model. Each governing document is its own entity:
`MasterTrust` (versioned) · `JoinderAgreement` per account, carrying `stateForm` and
`formVersion` because different accounts used different state forms and the forms are not
interchangeable · optional `Addendum` · optional `CourtOrder`.

An `AccountRecord` **references** its document set. It never flattens it.

### 4b. Every substantive field carries its provenance

The most important decision in the project. Do not skip or simplify it.

```ts
export interface SourceRef {
  documentId: TrustDocumentId;
  page: number;
  locator?: string;      // "§4.2(b)", or a bounding box once OCR is real
  quotedText?: string;   // the words the value was read from
}

export interface Provenanced<T> {
  value: T;
  source: SourceRef | null;            // null = not yet located in any document
  confidence: 'high' | 'medium' | 'low' | 'unread';
  enteredBy: 'extractor' | 'human';
  enteredAt: string;                   // ISO 8601
}
```

This is what turns "conflicts flagged, never silently resolved" into a real mechanism: when
two documents disagree you hold both values *and* both citations, and can show Will exactly
which page each came from. It is also what makes a memo defensible in a legal file, and what
lets the OCR engine be swapped later without touching the domain.

### 4c. The account record

Model every one of these:

- Beneficiary identity, DOB, **derived** current age, state form used
- Grantor, relationship, contact, KYC status
- Funding amount, source of funds, current balance
- Distribution standard elected; whether disbursements are **court-blocked**
- **`addendumExpresslyIncorporated: Provenanced<boolean>`** — this is wrong on some real
  files, so it must be provenanced and independently verified. **Never** infer it from an
  addendum merely existing.
- Addendum entitlements: recurring allowances, capped/uncapped purchase rights, categorical items
- Age-of-final-distribution election, and whether it conflicts with the addendum
- Whether a court order exists, and whether the election *requires* one
- Derived trigger dates: termination, tier openings, right-vesting
- Any second pot (annuity) sitting **outside** the trust
- Administrative gaps: account number, W-9, investment policy statement, fee schedule
- Distribution history

Derived values are **pure functions over the record, never stored fields** —
`deriveCurrentAge`, `deriveTerminationDate`, `deriveTierOpenings`, `deriveRightVesting`.
Each gets unit tests with hand-calculated expected values, including leap-year and
end-of-month boundaries.

### 4d. Findings — the conflict and gap engine

```ts
export interface Finding {
  id: FindingId;
  accountId: AccountId;
  severity: 'conflict' | 'gap' | 'info';
  code: string;            // stable, greppable: 'ADDENDUM_NOT_INCORPORATED_BUT_ENTITLEMENTS_CLAIMED'
  message: string;         // plain English, written for Will
  affectedFields: string[];
  sources: SourceRef[];    // every document involved in the disagreement
  status: 'open' | 'escalated' | 'resolved_by_human';
  resolution?: { actor: string; note: string; at: string };
}
```

Each rule is a pure function `(record) => Finding[]` in `trust-core/src/rules/` — one file
per rule, one test per rule. Start with at least:

- addendum entitlements claimed while `addendumExpresslyIncorporated` is false
- age-of-final-distribution election conflicting with the addendum
- an election requiring a court order where none is on file
- disbursements court-blocked but distribution history shows disbursements
- missing administrative items (account number, W-9, IPS, fee schedule)
- balance / burn-rate inconsistent with recorded funding

**A `Finding` leaves `open` only through an explicit human action carrying an actor
identity.** There must be no code path — none — that clears one automatically. If you find
yourself writing one, that is the bug.

### 4e. Adapters — interfaces here, fixtures in `apps/api`

Define as interfaces in `trust-core`, implemented in `apps/api`: `DocumentExtractor` ·
`AccountRepository` · `CustodialStatementSource` · `LedgerSource` · `ArchiveSource`.

All of them get a fixture-backed implementation. `DocumentExtractor` additionally gets a
real one — see §4f.

**`AccountRepository` is where a database goes later.** For now: a JSON-file-backed
implementation reading `fixtures/`. That satisfies the no-database rule while leaving the
CTO a clean seam.

### 4f. Document scanning — Google Gemini, the BrookHaven house standard

The source scans have no text layer, so extraction is real scope. BrookHaven uses **Google
Gemini** for document scanning on its other projects; match that, and do not introduce a
different engine.

**Before you write any of it, ask the CTO — through Will — for the exact Gemini model ID
and SDK package the other BrookHaven projects use, and match them.** Do not guess a model
ID from memory; they go stale and consistency with the existing projects is the actual
requirement here.

Build **two** implementations of `DocumentExtractor`:

- **`FixtureDocumentExtractor`** — reads hand-authored candidate output from `fixtures/`.
  Offline, deterministic. **This is the default, and it is what every unit test uses.** The
  green bar must never depend on a network call or an API key.
- **`GeminiDocumentExtractor`** — the real one. Sends page images and returns candidates.

The strong fit with §4b: **do not ask Gemini for raw text and then parse it.** Ask it for
structured output matching the account schema, and require it to return, for every field,
the **page number** and the **exact quoted text** it read the value from. Those map straight
onto `SourceRef`, so provenance is populated by the extraction itself rather than
reconstructed afterwards. Map model uncertainty onto the `confidence` field, and any field
it cannot locate must come back `confidence: 'unread'` with `source: null` — **never a
guess**. A confident wrong answer on `addendumExpresslyIncorporated` is the single most
damaging thing this tool could produce.

Everything Gemini returns is `enteredBy: 'extractor'` and lands in the intake review screen
for Will to confirm or correct. Nothing it extracts is ever treated as settled.

Rules for the integration:

- API key from an environment variable, read through config, **never committed** — `.env` is
  already gitignored, and `.env.example` documents the variable name with no value.
- Its integration test is skipped by default and runs only when the key is present. It runs
  against **synthetic scans**, never real ones.
- Handle the failure paths explicitly: rate limits, timeouts, malformed structured output,
  and partial extraction. A failed scan surfaces as a `gap` finding, never as a silent empty
  record.
- **In this prototype it only ever sees synthetic documents.** Pointing it at the real
  archive is a separate, CTO-authorized step — see §6.

## 5. Scope of v1 — build exactly this, then stop

**In:**
1. `packages/trust-core` — schema, derivations, rules, adapter interfaces, full unit tests.
2. `fixtures/` — **31 synthetic accounts**, deliberately varied: some with addenda, some
   without, some with court orders, several carrying the exact conflicts §4d detects, some
   missing admin items. The fixtures are the test surface — make them exercise the rules.
3. **Synthetic scans** — generate scan-like PDFs/images from a handful of those fixture
   accounts (`fixtures/**/*.synthetic.pdf`, already whitelisted in `.gitignore`). Make them
   realistically awkward: skew, noise, a signature block, a stamped exhibit number. These
   are what the Gemini extractor is developed and tested against. Clean synthetic documents
   prove nothing.
4. **`GeminiDocumentExtractor`** per §4f, plus the `FixtureDocumentExtractor` the test suite
   runs on.
5. `apps/api` — NestJS, feature-module-per-domain matching Haven's `apps/api/src` layout.
   Read endpoints for accounts and findings, plus intake-review write endpoints.
6. `apps/web` — Next.js 16 App Router:
   - account list with a findings-count column
   - account detail — every field showing its source citation
   - **intake review screen** — field-by-field confirm/correct against the extractor's
     candidate output, showing confidence
   - findings panel with an explicit human escalate/resolve action

**Out — do not build, even if asked mid-session:** the distribution-request memo generator ·
the calendar view · **extraction run against the real archive** (the extractor is built and
proven on synthetic scans only — pointing it at real documents is a separate CTO-authorized
step) · real Dropbox / QuickBooks / custodial feeds · auth beyond a hardcoded dev user ·
any database · any deployment or CI beyond typecheck/lint/test.

The intake **review surface** is the durable half of the workflow — it is what makes the
extractor's output safe to rely on, and it survives regardless of how the extraction is
done. Build it properly, not as an afterthought to the scanning.

## 6. Hard rules

1. **No database, no ORM.** If the work seems to require one, stop and escalate to the CTO.
2. **No real data, ever.** Records contain **minors' Social Security numbers, dates of birth
   and home addresses**. Never read the mounted shared archive. Never ask Will for a real
   document. Never copy real data into this repo — not even one file "just to test the
   parser." All fixtures are invented: fake names, fake addresses, fake SSNs.
   **This extends to the extractor: only synthetic documents are ever sent to Gemini from
   this repo.** Running extraction over the real archive is a CTO-authorized step outside
   this prototype — it is the moment real minors' data leaves the building, and it is not a
   decision that gets made mid-session by you or by Will.
3. **Full SSNs are never stored.** The schema holds `ssnLast4` only, as a branded type. If
   Haven later needs the full value, that lives in Haven's encrypted store.
4. **No PII in logs, URLs, route params, or query strings — ever.** Route by opaque
   `AccountId` UUID, never by name or SSN. Write a redaction helper, use it at every log
   site, and add a test asserting a record passed to the logger emits no DOB, no address,
   no SSN fragment. Verify that test actually fails when you deliberately log a DOB.
5. **Nothing auto-executes.** The tool drafts and flags; a person decides. Encode it as
   behaviour: generated artifacts stay `status: 'draft'`, findings clear only through a
   human action carrying an actor, and no rule silently picks a winner between conflicting
   documents.
6. **Reading aid, not legal advice.** Conflicts escalate, never quietly resolve. Export a
   single `DISCLAIMER` constant from `trust-core` and render it on every screen showing a
   derived conclusion.

## 7. Quality bar

- `pnpm typecheck && pnpm lint && pnpm test` clean before you tell Will anything is done.
- Every rule, derivation and date calculation has a test whose expected value you calculated
  by hand and can justify. **A test passing on a wrong expectation is worse than no test.**
- API endpoints get `supertest` coverage including failure paths.
- No `any`. No `@ts-expect-error`. No disabled lint rule without a comment explaining why.
- Self-check before each hand-off: `grep -rE "from '(react|next|@nestjs|express|node:fs)"
  packages/trust-core/src` must return nothing.

## 8. How to work with Will — read carefully

**Do not write application code yet.** Work in this order, stopping at each checkpoint:

1. **Setup (§0)** — toolchain, repo, guardrails, hook test. Your job entirely; Will does
   nothing. Flow straight from the end of §0 into step 2 without waiting to be told.
2. **Domain questions.** Ask Will your open questions about the *trust itself* — what a tier
   opening is, what makes an election require a court order, how the addendum interacts with
   the final-distribution age. Plain English. As many as you need. This is the first thing he
   is actually asked to do, so open it by saying so.
3. **The schema only** — `packages/trust-core/src/schema.ts` plus a plain-English summary of
   every field and every rule you intend to implement. **Stop and get his explicit
   approval.** This is the part he has to specify, cheap to change now and expensive later.
4. Then build §5 in order — **trust-core → fixtures + synthetic scans → Gemini extractor →
   API → web** — checking in after each, with the app runnable at every checkpoint.

The Gemini model ID, SDK and API key come from the CTO (§4f) — **request them through Will
early, in one message, so they arrive before you reach the extractor step.** If they have
not arrived by then, do not stall and do not guess a model ID: build against
`FixtureDocumentExtractor`, carry on to the API and web work, note it in
`.claude/handoff.md`, and slot the Gemini implementation in when the details land. Never ask
Will himself for a key.

At the end of every session, write `.claude/handoff.md` (what landed, what is stubbed, what
the CTO must wire up) and give Will a short prompt he can paste into a fresh session to
resume. When he asks "what did you assume that I haven't confirmed?", answer it honestly and
completely — that question is how he catches your buried guesses, and buried guesses about
this trust are the main way this project goes wrong.
