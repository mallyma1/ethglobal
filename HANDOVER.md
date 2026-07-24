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
| "You can't claw it back mid-scroll" | Vault is a Hedera account you funded but don't hold the key to. There is no cancel button to press when the dopamine hits. |
| "The money really moved" | N settled transfers on HashScan, one per second of scrolling. Not a database row saying we charged you. |
| "It stopped the moment you closed the tab" | The consensus timestamp of the final transfer. Nobody can backdate it, including us. |
| "You did five clean focus days" | A Hedera Consensus Service topic — ordered, timestamped, tamper-proof, portable. |
| "The bot moving my money is accountable" | World AgentKit proof that one real, unique, verified human stands behind the agent. |
| "The classifier isn't rigged to charge me more" | 0G TEE-sealed inference (stretch). |

**The attack this defeats:** every screen-time app has a misaligned incentive —
actually charging you risks churn, so enforcement quietly gets softer over time.
A vault the operator can't refund from, and a public log the operator can't edit,
cannot soften itself to keep you subscribed.

---

## The squad mechanic

Everyone in a squad stakes into a round and declares their intent and hours.
Drains redistribute to whoever is clean at that moment. Live leaderboard showing
who is bleeding into whose pocket. Settlement at the end of the round.

**Why Hedera specifically:** `TransferTransaction` supports atomic multi-party
transfers natively. One transaction, one second, debits your vault and credits
all teammates simultaneously, all-or-nothing. No contract, no loop, no
partial-failure state. Ethereum cannot do this without deploying a contract.

**Notably: zero Solidity anywhere in this build.** Vaults are native Hedera
accounts. That removes our biggest team gap (no confirmed contracts engineer)
and happens to qualify for Hedera's No-Solidity track.

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

**Chain** — `@hashgraph/sdk`, Hedera **testnet**, accounts from portal.hedera.com
- `TransferTransaction` — per-second drain, multi-recipient for squad mode
- `TopicCreateTransaction` / `TopicMessageSubmitTransaction` — HCS log of session
  start, each verdict, clean exit
- `ScheduleCreateTransaction` — pre-committed stake return, so getting your money
  back doesn't depend on the agent choosing to be nice later
- `AccountCreateTransaction` with `KeyList` — vault accounts
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

**Agent auth** — World AgentKit. Package name and API need verifying against
current docs; it was v0.1.5 limited beta as of March 2026. Spike first.

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
| 0–4 | **Spike the two risky SDKs and nothing else.** Can you fire a Hedera testnet transfer once per second reliably from Node? Can World AgentKit issue a proof? Both are load-bearing and both are new to us. Find out before building on top of them. |
| 4–9 | **The spine.** Extension → WS → backend → transfers start → tab closes → transfers stop. Ugly is fine. This alone is demoable. |
| 9–13 | **Classification.** Intent capture, per-page verdict, debounce and cache. This is the differentiator — protect this time. |
| 13–19 | **World AgentKit gate** around the authority to spend, plus Telegram onboarding. |
| 19–22 | **Squad mode.** Multi-recipient transfers, leaderboard state. |
| 22–28 | **Dashboard + sound.** The meter *is* the demo. Give it real design time. |
| 28–32 | **HCS logging**, then 0G or ENS only if genuinely free. |
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
| World AgentKit access | Sponsor rep at venue | Limited beta — chase early, it's the $8k track and the biggest schedule risk. |

---

## Open decisions

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
- **Estimated work (~41h) exceeds the window (~36h).** That's intentional — the
  cut lines are how it fits, not a plan we hope survives contact.
