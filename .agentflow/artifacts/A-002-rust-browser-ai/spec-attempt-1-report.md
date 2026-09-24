* _2026-09-24 17:00:48 (gpt-5.6-terra/high)_

# QKMJ Rust/Wasm browser remake design

## Purpose, release boundary, and journey

**Original Ask:** 「那麼用 Rust／WebAssembly 改寫, 讓它可以在瀏覽器運作, 同時加入 AI 玩家(可選強弱).」

R-1 is a local, single-browser four-seat game: one human and three bots, no account, server, network play, remote AI, persistence, PWA, deployment, or C-source change. The first page load may fetch static assets; play thereafter is local. A player chooses Weak, Medium, or Strong for each named bot (all Medium by default), starts a session, completes hands, selects Next Hand, or restarts.

The page is a Traditional-Chinese playable table, not a dashboard: jade felt fills the available page, the human's 16/17 tiles form the dominant lower edge, and opponents, discard rivers, melds, flowers, winds, dealer/consecutive-dealer and remaining public count surround it. Setup is a compact pre-hand strip; an in-table claim tray names only legal choices; a result sheet gives source, decomposition/tai, every settlement, and progression. Palette: `#155B4B` felt, `#0D3B32` felt-shadow, `#F5E9CF` ivory tile, `#B53B36` red suit, `#245A9C` blue suit, `#202522` ink. Use a restrained system Chinese display face (serif fallback) for winds/round labels and system sans for controls/data—no font download. Tile glyph/text, not colour alone, conveys identity. Desktop uses a square table; mobile stacks opponent summaries above a horizontally scrollable, still-labelled human hand. Real buttons, visible focus, keyboard equivalents, touch targets, status announcements, and `prefers-reduced-motion` are required. The distinctive but useful signature is the four directional discard rivers: each tile is read toward its owner, making turn and claim provenance legible. This avoids a generic card grid; animated tile motion is optional and never gates a turn.

Normal journey: choose bot strengths and Start; opening deal replaces flowers and marks the dealer's discard. On a human turn, selecting a tile (pointer, touch, or focused keyboard control) exposes only its legal discard/action; a claim pauses bots until the player chooses a named legal option or Pass. Bot status names its chosen draw/discard/claim/pass while the worker advances. Win or reserve draw exposes the result and Next Hand; Restart creates a new local session.

## Smallest design and paths

Create one Rust `cdylib`/`rlib` crate, reused by native tests and WebAssembly; avoid ECS, JS framework, bundler, server, persistence, network code, and an AI library. Proposed files:

| Path | Responsibility |
|---|---|
| `browser/Cargo.toml`, `browser/rust-toolchain.toml`, `browser/Cargo.lock` | pinned Rust/Wasm dependencies and toolchain (created only after availability check) |
| `browser/src/lib.rs` | narrow wasm exports and native crate entry |
| `browser/src/engine.rs` | tiles, seeded wall, phase machine, legal-action validation/application, snapshots |
| `browser/src/rules.rs` | winning decompositions, tai/exclusions, settlement and progression |
| `browser/src/ai.rs` | observation-only policies and seeded decision RNG |
| `browser/tests/engine.rs` | native deterministic rules/API checks |
| `browser/web/index.html`, `app.js`, `worker.js`, `styles.css` | semantic table UI, static ES module, one engine worker |

`worker.js` instantiates the generated wasm web module and owns `Game` plus bot stepping; it returns serializable snapshots and accepts versioned action messages. JSON strings via `serde`/`serde_json` through `wasm-bindgen` are the bounded, debuggable interface; do not add a bespoke protocol. Each snapshot has `game_id` and monotonically increasing `revision`; an action must match both, so stale/double clicks are rejected before mutation. Restart terminates and recreates the Worker, then ignores messages from its old `game_id`; that is smaller and safer than cancellation choreography. The UI never receives bot concealed hands before hand end. The engine provides `legal_actions(seat)`, `apply(action)`, public snapshot, and a private snapshot only for the requesting seat; AI receives a distinct `Observation` plus its legal actions, never `Game` or wall access. Shuffle RNG and per-policy decision RNG are separate and seedable.

The only added concepts are necessary boundaries: `Observation` makes the AI privacy contract testable; revision/game IDs make asynchronous UI input safe; the one Worker prevents bot turns from blocking rendering. Smaller alternatives—passing `Game`, accepting unversioned actions, or running bots on the main thread—violate those requirements. There is no general event bus, state store, or request layer.

## Rules profile (shown in help as “QKMJ legacy profile”, not universal Taiwanese mahjong)

Use the observed 144 tiles: 1–9/11–19/21–29 ×4, winds 31–34 ×4, dragons 41–43 ×4, flowers 51–58 ×1. Deal 16 consecutive tiles to seats 1–4, dealer receives one extra; assign door winds randomly each hand as legacy semantics. Opening replacement is dealer's extra first, then players in dealer order; all flower and kong replacements draw from the front wall. Preserve exposed melds, flowers, discards, seat/round wind, dealer and consecutive dealer.

An ordinary winning hand is five melds plus one pair after exposed melds. Tai inventory: 1 tai—dealer, closed, self draw, all simples, one cup, kong replacement, last-wall draw/discard, each wind/dragon, flower; 2—four-season/four-gentlemen flowers, all-chows, mixed terminal, mixed triple chow, straight, triple pung, three-kong; 3—closed self draw; 4—all pungs, half flush, pure terminal, mixed terminal/honours, small three dragons; 6—four concealed pungs/four kongs; 8—big three dragons, small four winds, full flush, all honours, five concealed pungs, pure terminals; 16—big four winds, heavenly/earthly/human win; consecutive dealer is `2*cont_dealer`. Preserve active QKMJ tai #0–7, #9–16, #21–27, #31–37, #39–43, #45–47, #49–52 at those values, their indirect entries (#17–20 double winds, #28 two cups, #29 three concealed pungs, #30 three kongs, #38 four concealed pungs), all coded exclusions/stacking, and the first maximum-total decomposition (strict `>`; no cap). Do not implement no-behaviour stubs: robbery kong #8, seven-grab-one #44, eight-fairies #48; nor special seven pairs/thirteen orphans/minimum-tai gates. Defaults: 20,000 points, base 500, tai 200. Self draw charges each other seat `500 + tai*200`, except a dealer payer uses `500 + (tai + 1 + 2*cont_dealer)*200`; discard win charges only discarder with that same dealer-payer surcharge. A dealer winner gains dealer tai. Non-dealer win moves dealer/clears count; dealer win retains dealer/increments; dealer wrap advances round wind.

After discard, collect legal replies: win then kong then pong then chow; simultaneous wins select first clockwise after discarder. Chow is next live seat only, suited only, with every valid shape. Discard kong consumes three; own-draw concealed and added kong are offered; all replace from front wall. No rob-kong opportunity. Before **every** ordinary or replacement draw, retain the final 16 tiles; if it cannot draw, end as wall draw, reveal/settle progression once, retaining dealer and incrementing consecutive dealer. This guards paths the C client missed.

### Proposed correction approval

Approve this safe preservation rather than emulating memory errors: score North double-wind at the current decomposition, iterate exposed melds correctly for all-honours, test suited tiles correctly for mixed terminals/honours, and award kong-replacement tai only when the actual winning source is a kong replacement (never flower replacement). Rust enums/vectors plus validated payloads replace fixed-array/message-buffer hazards. These changes preserve the intended active profile; they are not new scoring rules. Owner Design Go confirms this block and the random door-wind legacy behaviour.

## AI and invariants

Weak randomly chooses a bounded legal discard/claim set and may pass non-winning claims. Medium ranks its own hand by grouping/readiness. Strong compares a bounded short list using the same hand heuristic plus public discard/meld danger. No policy promises win rate. It may read public remaining count, public state/history and its own concealed hand, but not wall identity/order, opponents' hands, or hidden remaining information.

| Invariant | Enforced by | Evidence |
|---|---|---|
| INV-1 valid phase/action only; invalid input unchanged | `engine` transition gate | rejected-action snapshot equality |
| INV-2 exact tile, deal, flower, wall-16 lifecycle | `engine` wall/deal functions | seeded flower chain and reserve draw |
| INV-3 claims/kongs and priorities are legal/deterministic | `engine` legal actions | three chow shapes, all kongs, multi-claim fixtures |
| INV-4 legal five-meld/pair scoring, tai and settlement match profile | `rules` | each active tai/exclusion, maximum decomposition, both payment formulas |
| INV-5 no bot/view hidden-state leak | separate `Observation`/snapshots | compile/API assertions and serialized-view inspection |
| INV-6 versioned UI input cannot replay stale action | worker/engine revision check | duplicate/stale action fixture |

R-1: Purpose/architecture/INV-5; R-2: paths/build; R-3: journey/AI; R-4: journey/INV-1/6; R-5: Rules/INV-2–4; R-6: Rules/INV-2. These invariants are release gates, not documentation only.

## Slices, red-first evidence, and acceptance

1. Pin an available isolated Rust toolchain and matching Wasm packages; add tile/phase/action types. Start red with deterministic tile/deal/flower/reserve and invalid-action tests, then make native tests pass.
2. Add claims, kongs, decomposition/scoring/settlement/progression. Start red with all listed fixture categories, then implement exactly the profile.
3. Add observation-only policies. Start red with structural no-hidden-state tests and seeded fixtures where Weak/Medium and Medium/Strong each select a different valid action.
4. Add wasm boundary, Worker, and static accessible table. Start red with revision/restart tests and a browser exercise covering seeded complete win, wall draw, click/touch/keyboard, focus, legal claim/pass, bot progress, Next Hand and Restart.

Before selecting versions, record available `rustup toolchain list`, `rustup target list --toolchain <candidate>`, and compatible `wasm-bindgen` package/CLI release; pin those verified values in `browser/rust-toolchain.toml`, `Cargo.toml`, and lockfile without changing the global default. Thereafter the reproducible commands are: `cd browser && cargo test`; `cd browser && cargo build --release --target wasm32-unknown-unknown`; `wasm-bindgen --target web --out-dir web/pkg target/wasm32-unknown-unknown/release/qkmj_browser.wasm`; and `cd web && python3 -m http.server 8080`, then open `http://localhost:8080`. The implementation slice also records the exact isolated bootstrap command using the pinned toolchain (`rustup toolchain install <pinned>; rustup target add wasm32-unknown-unknown --toolchain <pinned>`) and matching pinned `wasm-bindgen-cli`; placeholders are deliberately not guessed in this design. Web target needs HTTP serving and no bundler; Worker computation cannot touch the DOM.

Risk: exhaustive legacy scoring is the largest correctness surface, so no score simplification is permitted. Deterministic seeds prove correctness; timing only proves responsiveness. Static asset availability after initial load is not offline/PWA support. No remotes/credentials/provider isolation is implied by this clone.

## Pending Design Go

This report is the proposed plan, not implementation authorization under Agentflow. Design Go is required to confirm the correction block, verified pinned tool versions, and this exact rules profile before creating source/config/dependencies. Rejected smaller alternatives: a draw/discard-only demo fails R-5; JS rules duplicate the testable Rust core; main-thread bot loops risk rendering; exposing one full game snapshot leaks hidden state; a generic score subset silently changes the contract.

Self-check: accepted requirements and corrected codewalk evidence frozen before specification.
