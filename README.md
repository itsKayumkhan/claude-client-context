# client-context

A Claude Code skill that turns raw client chat into a versioned project knowledge base.

Not a chat summarizer. It builds a set of Markdown files that a developer, designer, PM, or another AI can trust as the source of truth for the project, and it keeps them honest across months of conversation: what is confirmed right now, what changed, what was rejected, what is still pending, and why each decision was made.

## The problem

Client requirements arrive as chat. Three months in, the answer to "what did we agree on for payments?" is buried in 400 messages across two apps, and the answer you remember is usually the one from six weeks ago that the client already changed.

Generic summarization makes this worse. It flattens a rejected idea and a confirmed decision into the same bullet list, quietly picks a winner when two messages disagree, and invents a deadline from "sometime next week".

## What it does instead

- **Separates confirmed from everything else.** Every item carries one status: `CONFIRMED`, `NEW`, `CHANGED`, `PENDING`, `REJECTED`, `SUPERSEDED`, `ASSUMPTION`, `CONFLICT`. A client question is not a decision. A developer suggestion is not client approval.
- **Diffs against the existing context** on every run and reports what actually changed, with the previous value, the new value, and the impact.
- **Never silently resolves a conflict.** If two statements disagree and the chat does not settle it, it is flagged, not guessed.
- **Never rewrites history.** Superseded decisions move to `decisions.md` and `change-log.md` instead of disappearing.
- **Never invents** dates, years, budgets, or details, and never stores credentials.
- **Knows chronology.** A later message in the same batch is a change, not a conflict.

## Install

```bash
git clone https://github.com/itsKayumkhan/claude-client-context.git
cp -r claude-client-context/client-context ~/.claude/skills/
```

Restart Claude Code. For a single project instead of globally, copy it into `.claude/skills/` in the repo.

## Use

```
/client-context
```

Or just paste client messages and say "update project context". It also triggers on "client chat", "what changed since last chat", "add this to project context".

## Inputs

Two kinds, handled differently.

**Base chat.** The original conversation lives as a file at the project root: `.md`, `.txt`, `.pdf`, `.docx`, `.html`, `.json`, `.csv`. You never paste it. The skill finds it, reads it once, and records in `sources.md` what it ingested so later runs do not double count.

**Incremental messages.** Later changes pasted from Fiverr or WhatsApp. These are newer than the base chat by definition, so they win on any overlap. Both platform formats are parsed, including sender and timestamp.

Links are always kept, exactly as sent: the client's own site, design references, competitors, Figma, Drive, docs. Attachment names and sizes are kept as context. The actual image or file is never stored or opened, and its contents are never guessed.

## Output

```
docs/project/
├── project-context.md    # current state, the primary source of truth
├── project-rules.md      # rules future work must follow
├── requirements.md       # confirmed requirements by priority and area
├── decisions.md          # decisions with history and reasons
├── change-log.md         # newest first, every meaningful change
├── pending-items.md      # unresolved work and dependencies
└── sources.md            # participants, base chat file, ingestion log
```

Before writing anything, it prints a change report in the chat, so you approve the diff rather than discover it later:

```markdown
# Chat Analysis

## Sources Read
## Summary
## New Information
## Changed Information
## Conflicts
## Confirmed Decisions
## Rejected / Superseded Items
## Pending Items
## Important Deadlines
## Links & Attachments Received
## Impact on Existing Project
## Questions To Ask The Client
## Files Updated
```

`Questions To Ask The Client` is a ready-to-send list covering every conflict and every blocking pending item. If nothing meaningful changed, it says `No material project changes detected.` rather than inventing a diff from reworded text.

## Example

Input, a pasted message in the platform's own format:

```
A
acme-studio

15 Sept, 12:15
Morning, I should have all the feedback over in 48 hours.
Also let's go with Stripe instead of PayPal, and add CAD alongside USD.
Reference: https://example.com
```

Output extract:

```markdown
### Payment Provider Changed
- Previous: PayPal
- New: Stripe
- Status: CHANGED
- Source: Client, 15 Sept 12:15
- Impact: Payment integration, checkout, webhooks.

### Currency Changed
- Previous: USD only
- New: USD + CAD
- Status: CHANGED
- Impact: Pricing, checkout, product display.

## Pending Items
- Client feedback, promised 15 Sept 12:15 + 48h -> 17 Sept 12:15. Status: PENDING.
```

Note what did **not** happen: the 48 hour promise did not become a milestone, and the year missing from `15 Sept` was resolved from context rather than assumed.

## Status calibration

| Message | Status |
| --- | --- |
| "Yes, please add Apple Pay." / "Approved." | CONFIRMED |
| "Maybe we can add Apple Pay." / "Would be nice to have" | PENDING |
| "Can you do X?" | Question, not a requirement |
| "I should have feedback over in 48 hours." | PENDING, with derived date |
| "Remove Apple Pay." | REJECTED |
| "I don't love the green." (no replacement) | REJECTED for that option, plus a PENDING decision |
| A developer proposing an approach | ASSUMPTION until the client confirms |

## Source-of-truth priority

1. Latest explicit client confirmation
2. Latest explicit project or team decision
3. Earlier confirmed client decision
4. Existing project documentation
5. Developer or team assumption
6. Unconfirmed suggestion

Channel does not decide priority. Recency and explicitness do.

## Security

Passwords, API keys, tokens, private keys, OTPs, 2FA codes, session cookies, card numbers, and recovery codes are never written to these files. The skill records only that access is required and where to obtain it.

## Adapting it

`SKILL.md` is plain Markdown, edit it. The pieces most people change:

- **Input formats** under Inputs, if your chat comes from Slack, Upwork, email, or call transcripts rather than Fiverr and WhatsApp.
- **Status calibration table**, if your clients phrase approval differently.
- **Output location**, if you want something other than `docs/project/`.
- **Templates** in `client-context/templates/`, which are copied on first run.

## Contributing

Issues and pull requests welcome. Useful directions: parsers for other chat platforms, calibration examples for non-English client communication, and better conflict heuristics.

## Author

Built by **Kayumkhan Sayal**, a full stack developer from India who got tired of losing client decisions inside three-month-old chat threads.

<a href="https://kayumkhan-sayal.netlify.app/"><img src="https://img.shields.io/badge/Portfolio-00d9ff?style=for-the-badge&logo=netlify&logoColor=white" alt="Portfolio" /></a>
<a href="https://github.com/itsKayumkhan"><img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub" /></a>
<a href="https://linkedin.com/in/kayumkhan_sayal"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn" /></a>
<a href="https://twitter.com/kayumkhan_sayal"><img src="https://img.shields.io/badge/Twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white" alt="Twitter" /></a>
<a href="https://www.youtube.com/c/codemanoranjan"><img src="https://img.shields.io/badge/YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white" alt="YouTube" /></a>
<a href="mailto:kayumkhansayal2004@gmail.com"><img src="https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white" alt="Email" /></a>

- Portfolio: [kayumkhan-sayal.netlify.app](https://kayumkhan-sayal.netlify.app/)
- Email: [kayumkhansayal2004@gmail.com](mailto:kayumkhansayal2004@gmail.com)
- GitHub: [@itsKayumkhan](https://github.com/itsKayumkhan)
- LinkedIn: [kayumkhan_sayal](https://linkedin.com/in/kayumkhan_sayal)
- X / Twitter: [@kayumkhan_sayal](https://twitter.com/kayumkhan_sayal)
- YouTube: [codemanoranjan](https://www.youtube.com/c/codemanoranjan)

If this skill saves you a painful "wait, what did we agree on?" moment, a star on the repo is appreciated.

## License

MIT. See [LICENSE](LICENSE).
