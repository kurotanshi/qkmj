* _2026-09-24 16:55:40 (gpt-5.6-terra/high)_

# Browser Mahjong Remake: R-1 Contract

## Scope and requirement ledger

- **R-1 [confirmed]** Deliver a single-browser local game: one human and three AI players. No network multiplayer, accounts, server, external AI/API, ML training, deployment, or changes to the original C sources or `AGENTS.md`.
- **R-2 [confirmed]** Implement the game core in Rust compiled to WebAssembly, with a browser UI. After the initial page load, play has no ongoing server/API dependency. This does not require PWA installation, caching, persistence, or offline-first asset handling.
- **R-3 [confirmed]** Let the player set each AI to Weak, Medium, or Strong before a hand; default all three to Medium. The UI identifies each bot's strength.
- **R-4 [confirmed]** Support mouse, touch, and keyboard tile/action use. A human can select a tile and take only currently legal actions; the UI visibly states whose turn it is, legal responses, AI progress, result, and Next Hand/Restart.
- **R-5 [inferred, release boundary]** A hand is complete only after deal, flower replacement, draws, discards, all legal claims, wins, wall-exhaustion draw, scoring/settlement, dealer progression, and a next-hand path work. A draw/discard demo is not acceptable.
- **R-6 [inferred, preservation]** Retain the legacy 144-tile set: four each of 1–9 in three suits, four winds, four dragons, and eight flowers; 16 concealed tiles per player plus dealer's extra tile. Preserve visible discards, exposed melds, flowers, seat/round winds, dealer and consecutive-dealer state.

## Normal-user journey and acceptance

1. Open the already-loaded page, choose bot strengths, and start. The table deals, replaces flowers, shows the human hand and public state, and marks the active player.
2. On the human draw or a discard response, tiles and only legal action controls are operable by click/tap and keyboard; focus is visible and actions have text labels. Invalid actions cannot mutate the game.
3. AI turns advance without input and visibly report the chosen draw, discard, claim, or pass. Human claims pause progression until the player chooses a legal option or Pass.
4. A win shows winner, win source, score breakdown, per-seat settlement, dealer/round update, and Next Hand. Wall exhaustion shows a draw result and its required progression. Restart begins a new local session.

Acceptance: an automated engine check and a browser exercise each cover a seeded complete hand through win and through wall draw, flower replacement, chow/pong/kang (including added kong if retained), claim priority, score/settlement, Next Hand, restart, and every input method above. No score category may be omitted without an explicit owner-approved rule change.

## AI contract

All bots receive exactly their own concealed hand plus public state (their and others' exposed melds, flowers, discards, turn/action history, and configured seat/rule state). They must not inspect another concealed hand, wall order/content, or remaining-wall information.

- **Weak:** chooses among legal discards and claims with bounded randomness; it may pass non-winning claims.
- **Medium:** chooses legal discards/claims using its own hand's immediate grouping/readiness heuristic.
- **Strong:** considers the legal alternatives from the same information plus public discard/meld danger and short alternative-discard comparison.

Acceptance: fixed seeded states demonstrate at least one different selected action for each adjacent policy pair; all selected actions validate through the same rules engine; inspection tests prove no policy API exposes hidden hands or wall/remaining-wall data. These are behavior distinctions, not promised win rates.

## Legacy rule-preservation boundary

The observed client provides flower replacement; chow only to the next player and only in suited tiles; pong; concealed/discard/added kong paths; win; and nominal claim order win, kong, pong, chow. It calculates candidate 5-meld-plus-pair combinations, selects the maximum tai total, applies base/tai payments, dealer adjustments, and consecutive-dealer handling. On the final 16 wall tiles it declares a draw and starts another hand.

Before implementation, codewalk the legacy C and record a testable rule map: tile codes/multiplicities; dealing and replacement order; exact multi-claim tie-break; every kong and replacement path; legal winning-hand shapes; wall-draw/dealer progression; all settlement formulas; and all tai categories, values, exclusions, and stacking. This is a release gate, not permission to silently simplify scoring. `chkmake.c` declares 53 tai entries but several checks are empty or not invoked, while some implemented checks contain suspect indexing/conditions; determine whether each is intentional legacy behavior, a defect to reproduce, or needs the owner's ruling. Also resolve the client/server split where message flow, rather than one function, decides claims and draw timing.

## Smallest excluded scope

No online table, chat, login, save/resume, rankings, analytics, mobile packaging, PWA/cache, configurable rule variants, tutorial, art overhaul, or AI win-rate benchmark. Propose any of these separately.

Self-check: owner scope and role boundaries frozen before dispatch.
