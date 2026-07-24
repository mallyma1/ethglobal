# ETHGlobal Lisbon 2026 — Session Summary

*Written to migrate context into a dedicated hackathon repo. Covers the full
ideation session: brief, process, idea bank, current decision, team status, and
open next steps.*

## The brief

Malcolm is entering ETHGlobal Lisbon 2026 (submission deadline: Sunday July 26th,
09:00 WEST, roughly a 48-hour build window) and wanted a hackathon idea.

Hard requirements set early on:
- **Must be on Ethereum** (L2s fine, e.g. Base).
- **No generic DeFi / trading / yield / degen-casino stuff.** Not because he dislikes
  degen (he explicitly said he's "a degen at heart" and has separate degen ideas
  sitting around), but because he didn't want *this* project to be another one.
- **Real-world usage.** Something a normal person with zero crypto knowledge would
  want to use after hearing one sentence.
- **Themes he's drawn to:** wellness/health, social good, fun/funny/cool, dogs,
  socializing, ADHD/focus, real-world assets (land registry, provenance-style ideas).
- **AI agents welcome and expected** — this is the team's actual strength.
- **Team:** small team, 2-4 people, later confirmed max 5.

## Process used

1. Ran a multi-lens ideation workflow (six lenses: fun/viral, ADHD/focus,
   RWA/civic, dogs/pets, social/IRL, wildcard mashups), each lens generating ideas
   filtered against: normal-person appeal, blockchain being load-bearing (not
   decorative), 48-hour buildability for a 2-4 person team, and a specific
   judge-facing demo moment.
2. The workflow hit a mid-run spend limit; two of six lenses (ADHD/focus and
   fun/viral) completed before it died, yielding 8 fully-fleshed ideas. The other
   four lenses (RWA/civic, dogs-only, social/IRL, wildcard) never ran and were not
   separately generated; Pawport and Wild Goose (below) happen to cover dogs and
   social/IRL territory anyway.
3. Cross-checked against the live ETHGlobal Showcase (most recent event: New York
   2026) and general 2025 winner patterns. Finding: the current meta is saturated
   with AI-agent infra (ERC-8004 reputation, x402 micropayments, agent
   escrow/marketplaces) and the usual DeFi. Wellness, pets, IRL social, and civic/RWA
   are under-represented — confirming the chosen lane is open. Two buzzy
   consumer/fun projects worth knowing (comedy + human beats infra in memorability):
   **AIshley Madison** (jealous AI girlfriend judging your screenshare) and
   **Tea on Chain** (anonymous onchain dating reviews).
4. Full idea bank (8 ideas, deep detail) was written to a standalone document:
   `ethglobal-lisbon-2026-ideas.md` (already delivered to Malcolm, should be copied
   into the new repo verbatim — it has the full onchain-mechanics tables described
   below).

## The 8 ideas (short form — see the ideas doc for full depth)

**Tier 1 (top picks):**
1. **Pawport** — dog gets an ENS name + onchain "good-boy resume"; dog walker only
   gets paid on a GPS-verified, dual-signed walk attestation (EAS). Fixes real
   Rover/Wag payout-dispute complaints and the siloed-microchip-registry problem.
2. **Wild Goose** — prompt-to-scavenger-hunt generator with a live, stranger-fundable
   prize pot (escrow + EAS proof-of-presence + ENS team subnames).
3. **Pinky Swear** — stake $20 on a promise in your group chat; an AI referee reads
   evidence at the deadline and rules PASS/FAIL as a public EAS attestation, payout
   routes to charity/self/or a nominated "nemesis" friend.
4. **Doomtax** — while doomscrolling, real money drains out per-second via a
   Superfluid stream, stopping the instant you close the tab. **This is the one
   Malcolm has committed to.** See below for the current refined version.

**Tier 2 (backups):**
5. **Focus Royale** — staked group focus-sprints, quitters' money splits to
   finishers, World ID-gated to stop sybil farming.
6. **Pact** — someone else (e.g. your mum) stakes money on *you* hitting a goal; an
   AI referee inspects evidence and releases escrow on PASS.
7. **Loot List** — todo list that pays real-money loot drops on task completion,
   using onchain randomness so the odds are provably not rigged.
8. **Touch Grass Club** — social stakes for going outside; photo+GPS proof, flakes
   get slashed to the group.

Every Tier 1 idea's doc entry includes a table of exactly what data lives onchain,
in what form (contract call / EAS schema fields), and which specific attack it
defeats if you swap it for a plain database — the common pattern across all four:
**the app's own operator is a plausible cheater, and onchain removes them as a party
who can quietly intervene.** That's the "why blockchain" answer to give judges.

## Current direction: Doomtax (v3, evolving)

**Core loop:** declare focus hours to a Telegram agent → agent opens a Superfluid
stream from your personal vault the moment you visit a blocklisted site during that
window → money drains per-second, visibly, in real time → closing the tab stops the
stream instantly → clean focus days mint an EAS streak attestation.

**v2 refinement — destination is now a user choice at setup ("pick your poison"),
not a fixed rule:**
- **Charity mode** — drains to a chosen cause. Safest, most judge-friendly, no one
  profits from your failure.
- **Vault mode** — drains to a time-locked vault under your own address, returned in
  30/60/90 days. Reframes the product as forced savings with urgency, not a fine.
  Likely the mode most people keep using after the hackathon ends.
- **Rival mode** — drains to a nominated friend. Funniest and most social, but
  carries "someone profits from my failure" optics; frame as opt-in/mutual (both
  people run a rival stream on each other) to keep it clean.
- Technical point worth keeping in the pitch: one Superfluid primitive cleanly
  supports all three destination types via just a different receiver address, no
  branching logic — a fiat version would need three separate integrations
  (payment processor, savings-lock feature, P2P transfer). Good "Technicality"
  talking point.
- Demo plan: lead with charity mode as the safe, universal judge demo; show
  vault/rival modes in the last ~20 seconds of the video as "and it also does this."

**v3 addition — World AgentKit:**
- World (formerly Worldcoin) launched **AgentKit** in March 2026: an SDK letting an
  AI agent present a cryptographic, privacy-preserving proof that one real, unique
  verified human stands behind it (built on World ID + x402). Explicitly supports
  Claude Code as an agent.
- Fits Doomtax directly: the Telegram agent that negotiates focus hours and
  autonomously opens/closes the real-money Superfluid stream currently has no way to
  prove it's acting for one accountable human rather than an unaccountable bot. One
  World ID verification gives the agent that cryptographic backing, letting it act
  with standing permission to move funds without re-approval every focus block.
  This is "human-verified autonomy" in the exact phrase World uses for it, and
  directly answers the obvious judge question: "wait, your bot moves your money on
  its own?"
- Adds a third clean partner-prize track (World) alongside Superfluid and Base.
- Real risk to flag: AgentKit is brand new (public package first tagged March 6
  2026, v0.1.5, limited beta) — same caution as the Superfluid integration itself.
  Spike both on day one, keep a fallback (agent operates under a regular Privy
  wallet) if either SDK fights the team under time pressure.
- Sources referenced during research: world.org AgentKit announcement, Unchained
  and emelia.io explainer pieces (all findable via web search if the new repo needs
  fresh links).

**Not yet decided / open threads on Doomtax:**
- Exact blocklist mechanism and strictness (Chrome extension detecting active tab
  domain — confirmed direction, not yet spec'd in detail).
- Whether there's a mobile companion angle, or desktop-only for the hackathon.
- End-to-end wiring diagram of AgentKit + Superfluid + EAS (mentioned as a next
  step, not yet written).
- No full 48-hour execution plan has been written yet (architecture, exact
  contracts, team split, hour-by-hour timeline, demo script) — Malcolm explicitly
  said "not yet, refine the idea first" when offered this.

## Team status

- Team cap: **max 5** (including Malcolm).
- Malcolm is open to **either leading a team or joining one** — he was deliberately
  humbled in his outreach messaging, framing himself as someone who's spent time
  around L1s/L2s in operations (seed grants, internal automation, tooling) and only
  the last ~9 months building hands-on ("vibe-coding": agent fleets, bots, on-chain
  verification, payment rails), not a veteran engineer.
- Explicitly **not** shading DeFi/degen culture — he considers himself a degen too
  and has separate degen ideas on the side; this project's direction is a deliberate
  choice, not a value judgment on trading/DeFi builders.
- Gap analysis for what he needs in teammates, based on the shortlisted ideas: (1) a
  **smart-contract / onchain-integration engineer** with real prior shipping
  experience (EAS, ENS, Superfluid, Foundry/Hardhat, wagmi/viem) — Malcolm's own
  Solidity is modest; (2) a **frontend/UI engineer with strong design taste** to
  make the live demo (draining meter, verdict screens, spectator maps) feel polished
  — Malcolm's strength is backend/bots/ops, not UI polish.
- From the ETHGlobal Discord `#find-a-team` channel, candidates identified and
  reached out to:
  - **rxShri99** — full-stack, Solidity + Rust, has been building multi-agent AI
    systems. Best fit for the contracts role.
  - **leah** — full-stack dev + privacy journalist, strong design/UI/aesthetic
    instinct. Best fit for the frontend/UX role.
  - **serg_plusplus** — 8+ years web3, CTO of a crypto wallet, security/infra
    focus. Flagged as a strong backup/4th, especially valuable if real money flows
    end up more complex (escrow + Superfluid).
  - **laura** — Head of Tech at a multi-agent AI infra startup; background in
    applied AI, distributed systems, LLM agents, RAG, computer vision. Flagged as a
    possible 5th, her CV background is directly useful for Pawport-style
    photo-verification tasks and could co-pilot the agent/bot layer if Doomtax's
    scope grows.
  - A fifth poster (name not captured, mentioned wanting "hardware stuff" and crazy
    ideas) was also acknowledged in the humble group outreach message, no specific
    role identified yet.
- Two personalized DM drafts were written (to rxShri99 and leah) and a final,
  humbled group message was posted to `#find-a-team` explicitly stating he's open
  to starting *or* joining a team, name-checking everyone who'd already posted.
  **Status as of this summary: outreach sent, no confirmed teammates yet** — awaiting
  replies.

## Immediate next steps (pick up here in the new repo)

1. Decide whether to keep refining Doomtax's mechanics (blocklist strictness, mobile
   question, full AgentKit+Superfluid+EAS wiring diagram) or move to a full 48-hour
   execution plan.
2. Once 2-3 teammates confirm, lock final team roster and roles.
3. Write the full execution plan when ready: architecture, exact contract/EAS
   schema, stack choices, team split, Fri-evening → Sun-morning timeline, demo video
   script (2-4 min, ETHGlobal rules: no AI voiceover, 720p+, no rushing).
4. Decide partner-prize targets to formally apply for at submission (up to 3):
   current leaders are Superfluid, Base, World, with EAS/ENS as secondary
   integrations depending on which Tier-1 idea elements get folded in.
5. If time allows, revisit the Tier 2 backups or the never-generated RWA/civic and
   pure-dogs/wildcard lenses in case Doomtax hits a hard blocker during the build.

## Files to carry over into the new repo

- `ethglobal-lisbon-2026-ideas.md` — full idea bank with onchain-mechanics tables
  (already delivered to Malcolm as a file).
- This summary.
