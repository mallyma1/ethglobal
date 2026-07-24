# Doomtax

**While you doomscroll, your money visibly drains out of your account per second — and it stops the moment you close the tab.**

Built for ETHGlobal Lisbon 2026.

## The problem

Screen-time apps nag and get ignored because the cost of one more scroll is abstract and delayed. ADHD time-blindness means consequences that arrive next week don't exist. The cost needs to be immediate, visible, and continuous — like a taxi meter running.

## The onchain mechanic

Superfluid streaming money on Base. Park a stake (~25 USDC) in a small vault, declare focus hours with your agent over Telegram. A Chrome extension detects blocked sites during those hours; the backend opens a Superfluid stream from your vault to a pre-chosen charity, draining in real time until you leave the site. Clean exits and completed focus days earn EAS streak attestations.

## Why not a database

Per-second streaming money is a native crypto primitive with no fiat equivalent. Fiat versions (Forfeit, BePresent, Beeminder) are delayed batch charges where the company keeps the money and could quietly waive or reverse it. Here the drain is real, continuous, publicly verifiable, and goes to charity, not a house.

## Exactly what's onchain, and why it has to be

| Data | Where it lives | Why not just Postgres |
|---|---|---|
| The vault | Small vault contract holding your staked USDC | Needs to be a place you genuinely cannot instantly claw back mid-scroll (a "cancel" button in a normal app defeats the entire mechanism the second the dopamine hit of the site kicks in). Onchain custody removes the one-click undo. |
| The drain itself | A live Superfluid stream (per-second flow rate) from vault → charity address | This is the actual novel mechanism — no fiat rail does continuous real-time value transfer; the closest fiat equivalent is a delayed batch charge (Beeminder charges you once, after the fact). The visceral "meter ticking down live" only exists because Superfluid lets money move continuously and provably, not in a lump the app decides to apply later (and could quietly not apply). |
| Stream stop condition | Extension detects tab-close, backend calls `deleteFlow()` | The stop is instant and verifiable on the explorer in the same block, versus a database flag that says "user closed tab" which nobody but the app can verify happened, or when. |
| Streak record | EAS attestation per clean focus day | Same portability logic as the others: the record of your discipline isn't trapped inside one company's growth metrics, it's yours to show anywhere (a future insurer, employer wellness program, whatever). |

**The attack this defeats:** an app-side "just refund me" escape hatch, and a company quietly not-charging you because enforcing your own bet against yourself is bad for retention (every screen-time app has this misaligned incentive: annoying the user with a real charge risks churn, so enforcement quietly gets softer over time). A neutral vault + streaming primitive can't soften itself to keep you subscribed.

## 48h MVP scope

- Chrome extension reporting active-tab domain against a blocklist.
- Node/Express service opening/closing Superfluid streams via SDK.
- Privy embedded wallet + one small vault contract on Base Sepolia.
- Telegram agent for onboarding ("what are your trigger sites, what hours, what charity hurts the right amount").
- Big dashboard with a live draining counter.
- Desktop browser only, one charity, hardcoded rates.

## Demo moment

Judge is handed the demo laptop, funded with a stake, invited to open Twitter. A huge on-screen meter drains their balance cent by cent with an audible tick, the Superfluid stream is live on the explorer. They close the tab, the drain stops, the agent sends a dry one-liner to Telegram: *"That scroll cost you 0.43. The dogs' shelter thanks you."*

## Fun factor

A taxi meter for your attention. The agent's deadpan post-scroll receipts are extremely shareable.

## Risks

- Detection is client-side and bypassable via phone (concede openly — the point is friction, not prison).
- Superfluid SDK is new to the team; timebox a day-one spike, fall back to per-minute micro-transfers if streams fight back.
- BePresent already does fiat screen-time stakes; differentiation must stay on real-time streaming, verifiability, and charity routing.

## Partner prize targets

Superfluid, Base, Privy.

## Docs

- [`docs/session-summary.md`](docs/session-summary.md) — full ideation session history: brief, process, idea bank, team status, and open next steps.
