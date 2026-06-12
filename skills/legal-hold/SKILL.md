---
name: legal-hold
description: 签发、刷新、解除或报告证据保全通知 — 起草保全通知为 .docx，更新 _log.yaml 中的 legal_hold 字段。使用场景：用户要求"签发保全""刷新保全""解除保全"或需要全组合保全状态报告。
argument-hint: "[slug] [--issue | --refresh | --release | --status]"
---

# /legal-hold

1. 若 `--status`（无 slug）：读取 `_log.yaml`，生成全组合保全报告。
2. 否则：加载案件 context。
3. 加载 CLAUDE.md → 保密标记、保全模板指针。
4. 按标志路由：
   - `--issue`：记录范围、保管人、日期范围、系统。起草保全通知 .docx。更新 legal_hold 字段。
   - `--refresh`：记录范围/保管人变更。起草下一版本。
   - `--release`：记录解除日期、保留指示。起草解除通知。
5. 确认后写入。展示草稿通知和日志变更。

---

# 证据保全

## 目的

中国法律体系下的证据保全制度（《民事诉讼法》第81条、《最高人民法院关于民事诉讼证据的若干规定》），分为诉前保全和诉中保全。本 Skill 管理：**签发 → 刷新 → 解除 → 追踪**。

## 法域假设

证据保全义务依据中国法律。核心法律依据：《民事诉讼法》第81条（诉前/诉中证据保全）、《最高人民法院关于知识产权民事诉讼证据的若干规定》等。确认适用规则后起草。

## Load context

- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/_log.yaml` — log row (legal_hold fields + status)
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/matter.md` — matter context (counterparty, facts, key custodians from internal_owners)
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` — house style for litigation hold template pointer, privilege marking, escalation norms

**Conflicts gate — unbypassable.** Before issuing, refreshing, or releasing a hold, check `_log.yaml` for the matter slug. If the matter is not in `_log.yaml`, refuse and route:

> "I don't see [matter slug] in the matter log. Run `/litigation-legal:matter-intake` first so the conflicts check runs and the matter workspace is set up. I won't issue, refresh, or release a legal hold on a matter that hasn't been intaken — the conflicts check is the gate, and a hold issued against an unmanaged matter has no `_log.yaml` row to track `last_refresh` / `next_refresh` / `released` against."

Do not proceed on an unintaken matter. Intake is what runs conflicts and writes the `_log.yaml` row the `--refresh` / `--release` / `--status` flags operate against.

## Modes

The command takes a flag: `--issue | --refresh | --release | --status`. Default (no flag) → prompt.

### `--issue` — first issuance

Required when `legal_hold.issued == false` and the matter is active or reasonably anticipated.

**Before issuing the hold to custodians (the consequential act):** Read `## Who's using this` in `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md`. If the Role is Non-lawyer:

> Issuing a legal hold has legal consequences — the scope, custodian list, and timing create the preservation record the company will be judged on if 证据灭失 is argued later. Have you reviewed this with an attorney? If yes, proceed. If no, here's a brief to bring to them:
>
> [Generate a 1-page summary: the matter and trigger, the proposed scope and custodians, the forum-specific preservation rule researched, known 证据灭失 exposure, what could go wrong (too broad / too narrow), what to ask the attorney.]
>
> If you need to find a licensed attorney, solicitor, barrister, or other authorised legal professional in your jurisdiction: your professional regulator's referral service is the fastest starting point (中国律师协会或你所在法域的律师监管机构).

Do not send the notice without an explicit yes. Drafting and scoping do not require the gate — issuance does.

**Research the applicable preservation rule before issuing.** Identify the jurisdiction and the source of the preservation duty (common law, rule of civil procedure, regulatory preservation obligation, contractual). Confirm the currently operative trigger standard (when the duty attaches), scope standard (what must be preserved), and sanctions exposure (证据灭失 doctrine for the forum). Cite primary sources. Note that federal and state law can differ materially on trigger timing, scope, and remedy — flag the forum you're relying on. If uncertain, say so and get outside-counsel sign-off before issuing.

> **External deliverable:** the notice below is sent to custodians. Do NOT include a `PRIVILEGED & CONFIDENTIAL — ATTORNEY WORK PRODUCT — PREPARED AT THE DIRECTION OF COUNSEL` header on the outgoing notice; use the attorney-client marking in the template. Confirm the correct marking for your jurisdiction and matter.

**Inputs:**
1. **Scope** — categories of documents, data, communications. Start specific: contracts with counterparty, all communications referencing [project/subject], related financial records, calendar entries. `[SME VERIFY — scope too broad = operational burden; too narrow = 证据灭失 risk]`
2. **Custodians** — named individuals likely to hold responsive material. Pull suggestions from matter.md internal_owners and from common roles (business lead, HR partner if employment, CISO if data). `[SME VERIFY — the custodian list is the difference between defensible preservation and a gap argument]`
3. **Date range** — when to start preserving from (usually: triggering event or earlier), through the present + ongoing.
4. **Systems** — email, 飞书/企业微信, file shares, devices (including BYOD if applicable), 项目管理工具, CRM, legacy systems.
5. **Urgency** — if litigation already served or demand received with threat of suit, this goes out today.
6. **Effective date** — date of the hold.

**Draft the notice** to each custodian, using the house template in `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` if one is configured; otherwise the default template below.

**Default hold notice template:**

```
[PRIVILEGED & CONFIDENTIAL — ATTORNEY-CLIENT COMMUNICATION]

DATE: [effective date]
TO: [custodian name]
FROM: [signer — per `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` default]
RE: LITIGATION HOLD NOTICE — [matter short name]

You are receiving this notice because [company] has determined that [one-
sentence description of the dispute / investigation, avoiding prejudicial
detail]. The law requires preservation of documents and communications
potentially relevant to this matter.

EFFECTIVE IMMEDIATELY, you must preserve:

1. All documents, emails, text messages, 飞书/企业微信 messages, and other
   communications relating to [scope bullet 1].
2. [scope bullet 2]
3. [scope bullet 3]
...

This preservation obligation applies to:
- Email (including sent, archived, deleted folders)
- 飞书/企业微信/messaging platforms
- Shared drives and cloud storage
- Personal devices used for company business (BYOD)
- Paper documents
- Voicemails
- Calendar entries and meeting notes

DO NOT:
- Delete, modify, destroy, or dispose of any potentially responsive material
- Auto-delete or "Inbox Zero" any email or messaging

Coordinate with [legal contact] before sharing this notice with direct reports
or IT.

Direct questions about this notice or your preservation obligations to [legal
contact]. You may continue to discuss the underlying business subject matter
with colleagues as needed for your work, but do not discuss this legal notice,
the litigation, or legal strategy.

IF YOU ARE UNSURE whether something is covered, ERR ON THE SIDE OF PRESERVING.

Please acknowledge receipt of this notice by [reply / link / form] within
three business days. If you have questions, contact [signer email].

This notice remains in effect until you receive written notice of its
release. You may be asked to reaffirm compliance at periodic intervals.

[Signer signature block]
```

**Send gate (closing note on the draft):** Append to the in-chat preview of the notice — stripped before the notice goes to custodians:

> This is a draft legal hold notice for attorney review, not a notice ready to issue. Issuing a hold triggers preservation obligations the company will be judged on in any later 证据灭失 argument, and the notice itself may be discoverable. A licensed attorney reviews, approves, and issues. Do not distribute this draft unreviewed.

**Writes:**
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/legal-hold-v1.docx` via the `docx` skill
- Appends to `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/history.md`:
  ```
  ## [YYYY-MM-DD] — Legal hold issued

  Hold issued to [N] custodians: [list].
  Scope: [one-line summary].
  Next refresh: [YYYY-MM-DD (default issued + 6 months)].
  ```
- Updates `_log.yaml` row:
  ```yaml
  legal_hold:
    issued: true
    issued_date: [YYYY-MM-DD]
    scope: "[one-line summary]"
    custodians: [list]
    last_refresh: [YYYY-MM-DD]   # same as issued_date on first issuance
    next_refresh: [YYYY-MM-DD]   # default: issued_date + 6 months
    released: null
  ```

### `--refresh` — periodic reaffirmation

Refresh cadence: default 6 months; adjustable per matter. When `next_refresh < today` (or user invokes manually), the skill drafts a refresh notice.

**Inputs:**
1. Any **scope changes** since last refresh (new topics surfaced in 证据开示, new custodians, new systems).
2. Any **custodians to add or remove** (departures need special handling — see below).
3. Re-confirmation language.

**Refresh notice template:** similar to issuance; opens with "This is a reaffirmation of the legal hold originally issued [date]." Lists current scope (amended if needed). Requests re-acknowledgment.

**Departed custodians:** if a custodian has left the company since last refresh, the skill flags this as a preservation action item — the departing employee's files and email archive need to be preserved at IT level, not just via notice to the individual. Records this in history.md as a separate entry requiring action.

**Writes:**
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/legal-hold-v[N].docx` (next version number)
- `history.md` entry
- `_log.yaml`: updates `last_refresh` and `next_refresh` fields; modifies `custodians` list if changed

### `--release` — close the hold

Usually at matter close. Confirm the matter is truly over (not on appeal, not likely to reopen, statute of limitations passed on related claims).

**Before releasing the hold (the consequential act — preservation obligations resume normal retention):** Read `## Who's using this` in `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md`. If the Role is Non-lawyer:

> Releasing a legal hold has legal consequences — once released, custodians may begin deleting material. Release at the wrong time creates 证据灭失 exposure. Have you reviewed this with an attorney? If yes, proceed. If no, here's a brief to bring to them:
>
> [Generate a 1-page summary: the matter status, why release is proposed now, related-claim / appeal / SOL exposure, custodian impact, what could go wrong, what to ask the attorney.]
>
> If you need to find a licensed attorney, solicitor, barrister, or other authorised legal professional in your jurisdiction: your professional regulator's referral service is the fastest starting point (中国律师协会或你所在法域的律师监管机构).

Do not send the release notice without an explicit yes.

**Inputs:**
1. Confirmation of release authority (usually the signer or GC).
2. Release date.
3. Retention instruction — what happens to the material that was under hold? (Return to normal retention? Continue preserving for defined period? Transfer to archive?)

**Release notice template:** one paragraph, formal. "The litigation hold issued [date] regarding [matter] is released effective [date]. Normal retention resumes."

**Writes:**
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/legal-hold-release.docx`
- `history.md` entry
- `_log.yaml`: sets `released: [YYYY-MM-DD]`

### `--status` — report across the portfolio

Read `_log.yaml`. Produce a report:

```markdown
# Legal Hold Status — [today]

## Active holds

| Matter | Issued | Last refresh | Next refresh | Custodians | Status |
|---|---|---|---|---|---|
| [slug] | [date] | [date] | [date] | [N] | [ok / ⚠️ refresh due / ❌ overdue] |

## ⚠️ Attention

- **Refresh overdue:** [list slugs where next_refresh < today]
- **Refresh due within 30 days:** [list]
- **Matters active without hold issued:** [list — high/critical risk first]
- **Matters closed with hold still active:** [list — consider release]

## Recently released

[last 5 released holds with dates]
```

This is a separate command invocation (`/legal-hold --status` with no slug) OR invoked by `/portfolio-status` as a section in the portfolio rollup.

## Integration with portfolio-status

The `portfolio-status` skill already flags "Hold not issued on active litigation." This skill is what resolves those flags. Worth cross-referencing in the briefing when a matter is opened: if `legal_hold.issued == false`, `/matter-intake` closes by offering to run `/legal-hold --issue`.

## What this skill does not do

- **Enforce preservation.** It issues the notice; IT/custodians preserve. The skill flags when a custodian leaves (so IT can preserve at system level) but doesn't reach into systems.
- **Make scope calls alone.** The skill proposes scope from matter context; the user confirms. Scope too broad = operational burden. Scope too narrow = 证据灭失 risk. User's judgment.
- **Auto-refresh without review.** Even when `next_refresh` comes up, the user reviews scope changes before the refresh notice goes out.
- **Send the notice.** Drafts .docx; user sends via email per house convention. (Future integration: 邮件客户端 could send directly after user review.)
