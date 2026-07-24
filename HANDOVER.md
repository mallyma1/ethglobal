# Doomtax — Handover

ETHGlobal Lisbon 2026. Submission deadline **Sunday 26 July, 09:00 WEST**.

Drop this file in the new repo as `HANDOVER.md`. It's the single source of truth
for what we're building and why.

---

## One-liner

**Doomtax doesn't block Twitter. It taxes doomscrolling.**

You declare an intent to an agent ("I'm researching Hedera SDKs this afternoon").
While you browse, an AI classifier judges what you're actually reading. Read a
consensus thread — nothing happens. Scroll into celebrity drama — real money
starts leaving your staked vault, one settled transaction per second, into your
teammates' pockets. Close the tab and it stops dead.

Same site. Same session. Opposite verdicts.

---

## The problem

Screen-time apps nag and get ignored because the cost of one more scroll is
abstract and delayed. ADHD time-blindness means consequences that arrive next
week don't exist. The cost needs to be immediate, visible, and continuous — like
a taxi meter running.

Blocklists also fail for a second reason: they're binary. X is where a lot of
real work happens. An app that blocks the whole domain gets uninstalled by
Wednesday.

---

## Why this has to be onchain

Every claim maps to an artifact a judge can click:

| Claim | The artifact |
|---|---|
| "You can't quietly claw it back mid-scroll" | Stake is committed onchain — see the open decision on allowance vs vault account below. |
| "The money really moved" | N settled transfers on HashScan, one per second of scrolling. Not a database row saying we charged you. |
| "It stopped the moment you closed the tab" | The consensus timestamp of the final transfer. Nobody can backdate it, including us. |
| "You did five clean focus days" | A Hedera Consensus Service topic — ordered, timestamped, tamper-proof, portable. |
| "The bot moving my money is accountable" | World AgentKit proof that one real, unique, verified human stands behind the agent, plus framework-level policies bounding what it can call at all. |
| "The classifier isn't rigged to charge me more" | 0G TEE-sealed inference (stretch). |

**The attack this defeats:** every screen-time app has a misaligned incentive —
actually charging you risks churn, so enforcement quietly gets softer over time.
A stake the operator can't refund from, and a public log the operator can't edit,
cannot soften itself to keep you subscribed.

---

## The squad mechanic

Everyone in a squad stakes into a round and declares their intent and hours.
Drains redistribute to whoever is clean at that moment. Live leaderboard showing
who is bleeding into whose pocket. Settlement at the end of the round.

**Why Hedera specifically:** `TransferTransaction` supports atomic multi-party
transfers natively — confirmed in the Agent Kit as `TRANSFER_HBAR_TOOL`, which
takes `transfers: Array<{accountId, amount}>`. One transaction, one second,
debits your stake and credits all teammates simultaneously, all-or-nothing. No
contract, no loop, no partial-failure state. Ethereum cannot do this without
deploying a contract.

**Notably: zero Solidity anywhere in this build.** That removes our biggest team
gap (no confirmed contracts engineer) and happens to qualify for Hedera's
No-Solidity track.

---

## Prize targets

ETHGlobal caps submissions at **three partner categories** — confirm at the venue.

**Primary three (~$20k surface):**
1. **World — AgentKit New Use Cases ($8k).** An agent with standing permission to
   move real money on your behalf. Self-imposed behavioural finance is nowhere
   near their exclusion list (agent reputation / content gen / API discounts).
2. **Hedera — AI & Agentic Payments ($6k).** An agent executing real transfers on
   testnet. Literal description of what Doomtax does.
3. **0G — Best AI Product ($6k).** A model is deciding whether to take your money;
   sealed inference means the operator can't bias it. Genuine fit, not a bolt-on.

**Swap-ins if 0G doesn't land:**
- Hedera No-Solidity ($3k, split up to 3 teams, no booth requirement)
- ENS AI Agent Integration ($1.5k, single winner, **requires in-person booth
  Sunday morning** — only claim it if someone is confirmed to stand there)

**Not targeting The Graph or 1inch/Uniswap/Sui.** Doomtax consumes no indexed
chain data and has no swap step. Those tracks explicitly penalise bolted-on
integrations, so forcing one in scores worse than skipping it.

> Note: Base, Superfluid and Privy are **not** sponsors at Lisbon 2026. Earlier
> drafts of this pitch targeted all three. Ignore any doc that still says so.

---

## Hedera Agent Kit — what the fork gives us

Fork: `mallyma1/hedera-agent-kit-js` (upstream `@hashgraph/hedera-agent-kit`,
pnpm monorepo, Node >= 18).

**The critical architectural rule: two layers, deliberately separated.**

| Layer | Uses | Why |
|---|---|---|
| Per-second drain loop | Raw `@hashgraph/sdk` | Must be fast, cheap and deterministic. **Never put an LLM in this loop** — a model call per second is slow, expensive and non-deterministic. |
| Telegram onboarding agent | Hedera Agent Kit tools | Natural language in, Hedera transaction out. Exactly what it's for: "stake 25 HBAR and block X between 2 and 6". |

Getting this wrong is the single most likely way to burn a day.

**What we take from the kit:**

- **`TRANSFER_HBAR_TOOL`** — multi-recipient array confirms the squad split works
  in one atomic call.
- **`APPROVE_HBAR_ALLOWANCE_TOOL` / `TRANSFER_HBAR_WITH_ALLOWANCE_TOOL`** — the
  allowance path (see open decision).
- **`HcsAuditTrailHook`** — writes every tool execution to an HCS topic
  automatically. This is our session/streak log for free, and it's more
  defensible than hand-rolled logging because it can't be selectively skipped.
  Deletes ~2h of planned work.
- **`RejectToolPolicy` / `MaxRecipientsPolicy`** — hard limits enforced at the
  framework layer. Pairs with AgentKit for the strongest version of the World
  pitch: *the agent is human-verified **and** capability-bounded — it cannot call
  `delete_account`, cannot pay more than N recipients, and every action it does
  take lands on a public audit topic.*
- **`packages/mcp`** — the kit ships its own MCP server, so Hedera tools can be
  driven directly from Claude during development.

Relevant docs in the fork: `docs/HEDERATOOLS.md`, `docs/HOOKS_AND_POLICIES.md`,
`docs/MCP.md`, `examples/`.

---

## MCP servers

Already added to project config (`.mcp.json`). Note the docs server shows as
**pending approval** until approved in an interactive `claude` session.

**World Docs** — no auth, one tool (`search_world_documentation`):

```bash
claude mcp add --transport http --scope project world-docs https://docs.world.org/mcp
```

**World Developer Portal** — needs a team API key, and it is genuinely worth
setting up rather than clicking through the web portal at hour zero. It exposes
`create_app`, `configure_world_id`, `create_world_id_action`,
`get_world_id_signing_key`, `rotate_world_id_signing_key`,
`get_world_id_registration_status`, `get_app_config`, `get_team_context`,
`configure_mini_app`, `upload_app_image`, `submit_app_for_review`:

```bash
claude mcp add world-developer-portal \
  https://developer.world.org/api/mcp \
  --transport http \
  --scope project \
  --header "Authorization: Bearer api_..."
```

That means the entire World ID app + action setup can be done from the terminal
in minutes. Keep the key local — this server can mutate apps in the team.

---

## Stack

**Chrome extension** — Manifest V3, TypeScript, Vite
- `chrome.tabs.onActivated` + `onUpdated` for active-tab domain
- Content script on watched domains extracts visible post text via site-specific
  selectors (`article[data-testid="tweet"]` on X), not whole-page `innerText`
- WebSocket to backend with a **1s heartbeat**. Missed beats stop the drain —
  this handles tab close, crash, sleep and network drop identically, with no
  "did they really close it?" ambiguity

**Backend** — Node 20, TypeScript, Fastify + `@fastify/websocket`
- In-memory session state (single instance, 36 hours — Redis isn't worth it)
- `better-sqlite3` only if sessions need to survive restarts during demos

**Chain** — `@hashgraph/sdk` for the drain, `@hashgraph/hedera-agent-kit` for the
agent. Hedera **testnet**, accounts from portal.hedera.com.
- `TransferTransaction` — per-second drain, multi-recipient for squad mode
- HCS topic via `HcsAuditTrailHook` — session start, each verdict, clean exit
- `ScheduleCreateTransaction` — pre-committed stake return, so getting your money
  back doesn't depend on the agent choosing to be nice later (cuttable)
- HashScan testnet as the explorer pulled up mid-demo
- **Denominated in HBAR.** An HTS "test USDC" reads better on screen but forces
  token-association on every account — real friction for a cosmetic win

**Classifier** — Anthropic API behind a `Classifier` interface (so 0G swaps in
without touching call sites)
- `claude-haiku-4-5-20251001` for the per-page verdict — sub-second matters
- Forced tool-use for structured output:
  `{ aligned: boolean, category: string, confidence: number, oneLiner: string }`
- **Debounce hard:** classify on URL change, or every ~15s of dwell on the same
  URL, cached per URL per session. Never per second — that mistake makes this
  unshippable
- `claude-opus-5` for the Telegram receipt prose. Latency is irrelevant there and
  the writing is the shareable part

**Agent auth** — World AgentKit. It was v0.1.5 limited beta as of March 2026, so
verify the package and API against the docs MCP before building on it.

**Telegram** — grammY (better TypeScript DX than Telegraf)

**Dashboard** — Next.js 15, Tailwind, Vercel (a live link scores better than
localhost on several tracks). WebSocket for the meter, Web Audio API for the
tick, Framer Motion for the digit roll.

**Stretch** — 0G Compute via `@0glabs/0g-serving-broker`; ENS text records via viem

---

## Build order

~36 hours. Hard rule: if a layer isn't clean by its box, cut it rather than ship
it half-wired.

| Hours | Work |
|---|---|
| 0–4 | **Spike the two risky SDKs and nothing else.** Can you fire a Hedera testnet transfer once per second reliably from Node? Can World AgentKit issue a proof? Set up the World ID app and action via the Developer Portal MCP while you're here. Both SDKs are load-bearing and both are new to us — find out before building on top of them. |
| 4–9 | **The spine.** Extension → WS → backend → transfers start → tab closes → transfers stop. Ugly is fine. This alone is demoable. |
| 9–13 | **Classification.** Intent capture, per-page verdict, debounce and cache. This is the differentiator — protect this time. |
| 13–19 | **World AgentKit gate** around the authority to spend, plus Telegram onboarding on the Agent Kit tools, plus policies. |
| 19–22 | **Squad mode.** Multi-recipient transfers, leaderboard state. |
| 22–28 | **Dashboard + sound.** The meter *is* the demo. Give it real design time. |
| 28–32 | **HCS hook** wired, then 0G or ENS only if genuinely free. |
| 32–36 | **Submission.** README pointing to the code each sponsor cares about, per-sponsor docs, demo video. Several tracks grade clarity at 10% and every one requires a video — budget this properly. |

### Cut lines

- **Must ship:** spine, Hedera transfers, classification, dashboard. Complete and
  winnable on its own.
- **Ships if the spike lands:** World AgentKit. It's the $8k track so it beats
  everything below it — which is exactly why it's at hour zero.
- **Cut without hesitation:** 0G, ENS, scheduled return, SQLite persistence.
- **If nobody joins the team by Saturday morning:** drop squad mode, not
  classification. Classification is the differentiator; squad mode is a
  multiplier on an idea that already works.

### Known fallback

If 1/sec transfers prove flaky, settle every 3 seconds and animate the counter
smoothly between settlements. Judge sees a smooth drain; ledger shows real
transactions. **Decide this in the hour 0–4 spike, not at hour 30.**

---

## Open decision: allowance vs vault account

This is the sentence judges will push on, so decide it deliberately.

**Allowance (faster).** User keeps their own account and key, and grants the
agent a capped HBAR allowance. No account creation, no funding transfer, no key
handover — a judge's own account works directly. Saves roughly 4 hours.
*Weakness:* `DELETE_HBAR_ALLOWANCE_TOOL` exists, so the user can revoke
mid-scroll. The escape hatch we claim to kill is technically still there.

**Vault account (honest).** A separate Hedera account funded at setup whose key
the user does not hold, with a pre-committed `ScheduleCreateTransaction`
returning the remainder. No revoke path exists. Costs an extra custody flow.

**Recommendation:** build allowance, and make revocation *loud* — log it to the
HCS topic and surface it on the squad leaderboard. Then the honest pitch is
"revoking is a signed, public, socially visible act" rather than "it's
impossible", which is still far stronger than a database flag. If there's slack
on Saturday, upgrade to the vault account.

---

## Demo script

Judge is handed the laptop: *"You told it you're researching Hedera. Go read
Twitter."*

1. They land on a consensus thread. Meter sits at zero. Agent line appears:
   *"Fine. That's research."*
2. They scroll into celebrity drama. The meter erupts — ticking audibly — and on
   the leaderboard beside it, teammates' balances climb in real time out of the
   judge's pocket.
3. Pull up HashScan: transactions landing one per second, each splitting to
   multiple recipients atomically.
4. They close the tab. Dead stop. Point at the final transaction's consensus
   timestamp — that's the proof, not a database row.
5. Telegram buzzes: *"That was not research. That was a man arguing about
   football. 0.43 to the squad."*

The zero-to-erupting contrast on the same site, in about eight seconds, is the
moment. Everything else supports it.

---

## Credentials needed

| What | Where | Notes |
|---|---|---|
| Hedera testnet account ID + private key | portal.hedera.com | Free, instant. Blocks the hour-zero spike without it. |
| Anthropic API key | console.anthropic.com | For the classifier and receipts. |
| Telegram bot token | @BotFather | Two minutes. |
| World developer portal team API key | developer.world.org | Unlocks the Developer Portal MCP — World ID app and action setup from the terminal. |
| World AgentKit access | Sponsor rep at venue | Limited beta — chase early, it's the $8k track and the biggest schedule risk. |

---

## Open decisions

- **Allowance vs vault account** — see above. Highest-value decision in this doc.
- **Team roster.** Outreach sent to rxShri99 (contracts/full-stack), leah
  (frontend/design), serg_plusplus (infra), laura (AI/agents). None confirmed as
  of writing. First hire priority is frontend — the meter is the demo.
- **Third prize slot:** 0G vs Hedera No-Solidity vs ENS. Decide at submission,
  not now; build so all three stay possible.
- **ENS booth:** only claim that track if someone is confirmed free Sunday morning.
- **Squad size for the demo:** two-person is the same code and much easier to
  show than three-plus.

---

## Honest risks

- **Detection is client-side and bypassable via phone.** Concede this in the pitch
  before a judge raises it. The point is friction, not prison.
- **World AgentKit and 0G are both brand new to us**, which is why one is hour
  zero and the other is explicitly cuttable.
- **Classification + squad mode together are roughly a full extra person of
  work**, and the team is currently unconfirmed. See the cut lines.
- **The schedule fits, but only just.** The original estimate was ~41h against a
  ~36h window; the HCS hook and the allowance path together claw back about five.
  There is no slack — the cut lines are how this ships, not a plan we hope
  survives contact.
