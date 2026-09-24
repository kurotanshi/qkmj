* _2026-09-24 17:03:47 (gpt-5.6-terra/high)_

# QKMJ Rust/Wasm browser remake design

## Purpose and normal journey

**Original Ask:** 「那麼用 Rust／WebAssembly 改寫, 讓它可以在瀏覽器運作, 同時加入 AI 玩家(可選強弱).」

R-1 is a local, single-browser four-seat game: one human and three bots. It has no accounts, server, network play, remote AI, save/resume, PWA, deployment, or C/`AGENTS.md` changes. Static files may load once; after they are ready, gameplay makes no network calls. The human begins physically seated East and is the initial dealer East, with 20,000 points. The scoreboard is session points, not cash, and is not persisted. Each named bot defaults to Medium; its Weak/Medium/Strong selector can be changed before a hand and is immutable during that hand.

The normal journey is: select bot strengths and Start; the table deals, replaces flowers, and marks the dealer's discard. On a human draw or claim opportunity, clicking/tapping a tile or operating its focused keyboard button exposes only legal discards/actions. A named claim tray offers legal choices and Pass; bots pause for it, otherwise the Worker announces each bot's draw/discard/claim/pass. A win or reserve-wall draw shows source, decomposition/tai, per-seat settlement and progression, then Next Hand; Restart starts a new local session.

The page is a Traditional-Chinese mahjong table, not a dashboard: jade felt fills the page; the human's 16/17-tile hand dominates its lower edge; opponents, directional discard rivers, melds, flowers, winds, dealer/consecutive-dealer, and public remaining count ring it. The rivers orient tiles toward their owner, making a discard's source and a claim readable. Use `#155B4B` felt, `#0D3B32` shadow, `#F5E9CF` ivory, `#B53B36` red, `#245A9C` blue, and `#202522` ink; restrained system Chinese serif fallbacks label winds/rounds and system sans serves controls/data—no font download. Desktop is a square table; mobile stacks opponent summaries and keeps a labelled, horizontally scrollable hand. Tile text/glyphs, not colour alone, identify tiles. Controls are real buttons with visible focus, adequate touch targets, text status, and reduced-motion support. This is intentionally one playful table surface; cards, hero copy, and decorative animation are rejected.

## Smallest design and proposed paths

One Rust `cdylib`/`rlib` crate under `browser/` owns the rules and is reused by native tests and Wasm. A static HTML/CSS/ES-module UI uses one module Web Worker, which owns the Wasm game and bot stepping so rendering remains responsive. There is no server, JS UI framework, Node bundler, ECS, networking, persistence, event bus, state store, or AI dependency.

| Proposed path | Responsibility |
|---|---|
| `.gitignore` | Ignore `/browser/target/`, `/browser/.tools/`, and `/browser/web/pkg/` build artifacts. |
| `README.md` | Concise link to `browser/README.md`. |
| `browser/README.md` | Recorded verified versions, isolated bootstrap, exact test/build/serve commands, and browser check. |
| `browser/Cargo.toml`, `Cargo.lock`, `rust-toolchain.toml` | One crate, pinned dependencies/toolchain after availability verification. |
| `browser/src/lib.rs` | Native entry and narrow Wasm boundary. |
| `browser/src/engine.rs` | Tile/wall/phase state, legal actions, validated mutation, projections. |
| `browser/src/rules.rs` | Decomposition, tai/exclusions, settlement, dealer progression. |
| `browser/src/ai.rs` | Observation-only policies and seeded decision RNG. |
| `browser/tests/engine.rs` | Native seeded engine/rule/API checks. |
| `browser/web/index.html`, `app.js`, `worker.js`, `styles.css` | Semantic table, static module UI, Worker, and responsive accessible styling. |

The Wasm boundary accepts/returns plain versioned JSON via `wasm-bindgen` and `serde`; this is the smallest inspectable interface. It exports only a public projection plus the human's private projection, never arbitrary per-seat views; bot concealed hands appear only in the completed-hand reveal. Internally, `Game::legal_actions(seat)`, `Game::apply(action)`, and per-seat projections serve rules; an AI instead receives an `Observation` and legal actions. An action carries the snapshot revision and is rejected before mutation unless current. Restart terminates the old Worker, removes its callback, and creates a new Worker; Worker-instance identity discards old messages, so no separate `game_id` protocol is needed.

These added concepts each serve a requirement: `Observation` makes hidden-state fairness testable; revision prevents stale/double-click mutation; one Worker prevents bot work blocking paint. Smaller alternatives—giving AI `Game`, unversioned UI actions, or main-thread bot steps—break those needs. Shuffle RNG and policy RNG are separate and seedable.

## QKMJ legacy rules profile

Help labels this **「QKMJ legacy profile」**, not universal Taiwanese mahjong. Use the 144 tiles: 1–9, 11–19, 21–29 ×4; winds 31–34 ×4; dragons 41–43 ×4; flowers 51–58 ×1. Deal 16 consecutive tiles to seats 1–4; dealer receives one extra. Preserve the source's independently randomized scoring door-wind assignment on every hand, explicitly explained in help as legacy semantics even though the human's display seat and initial dealer are East. Preserve exposed melds, flowers, discards, seat/round winds, dealer and consecutive dealer.

Opening flower replacement is dealer's extra tile first, then each player in dealer order; flower and kong replacements draw from the front wall. An ordinary win is five melds plus a pair after exposed melds. Before **every** ordinary or replacement draw, enforce the 16-tile reserve; inability to draw ends the hand as a wall draw, reveals results, retains dealer, and increments consecutive dealer once. This deliberately repairs the source's unchecked replacement path.

After a discard, collect replies in win, kong, pong, chow priority. Simultaneous wins select the first clockwise seat after the discarder. Chow is only by the next live seat, only suited 1–29, and offers every legal shape; discard kong consumes three. Own-draw concealed and added kong are offered, each draws a front-wall replacement. There is no rob-kong opportunity. A numeric pong/kong tie need not be specified: valid four-copy tiles cannot support rival claims of the required multiplicity.

Preserve active QKMJ tai values, coded exclusions/stacking, and the first maximum-total decomposition (strict `>`, no cap):

| Tai | Active or indirect QKMJ entries |
|---:|---|
| 1 | #0 莊家, #1 門清, #2 自摸, #3 斷麼九, #4 一杯口, #5 杠上開花, #6 海底摸月, #7 河底撈魚, #9–#16 winds/dragons/花牌 |
| 2 | #17–#20 double winds (indirect), #21 春夏秋冬, #22 梅蘭菊竹, #23 全求人, #24 平胡, #25 混全帶麼, #26 三色同順, #27 一條龍, #28 二杯口 (indirect), #29 三暗刻 (indirect), #30 三杠子 (indirect), #31 三色同刻 |
| 3 | #32 門清自摸 |
| 4 | #33 碰碰胡, #34 混一色, #35 純全帶麼, #36 混老頭, #37 小三元 |
| 6 | #38 四暗刻 (indirect), #39 四杠子 |
| 8 | #40 大三元, #41 小四喜, #42 清一色, #43 字一色, #45 五暗刻, #46 清老頭 |
| 16 / variable | #47 大四喜, #49 天胡, #50 地胡, #51 人胡; #52 連莊 is `2*cont_dealer` |

Prohibited no-behaviour stubs are #8 搶杠, #44 七搶一, and #48 八仙過海: although called by the legacy scorer, their bodies add no rule. Seven pairs, thirteen orphans, special flower wins, and a minimum-tai gate are likewise absent. Defaults are 500 base, 200 per tai, and 20,000 starting points. Self draw charges each other seat `500 + tai*200`, except a dealer payer pays `500 + (tai + 1 + 2*cont_dealer)*200`; discard win charges only the discarder using that same dealer-payer surcharge. Dealer tai is already part of winner scoring. A non-dealer win advances dealer and clears the count; a dealer win retains dealer and increments it; dealer wrap advances round wind.

### Proposed correction approval

Design Go approves safe intended behavior, rather than reproducing defects: write 北風北 (#20) in the current decomposition; iterate actual exposed meld tiles for 字一色; classify suited tiles correctly for 混老頭; award #5 only for an actual kong-replacement win, never flower replacement; and use Rust enums/vectors plus validated payloads instead of fixed arrays/message buffers. These are corrections to active behavior, not added rules. No scoring subset may be substituted as an MVP.

## AI, invariants, and coverage

Legal wins are always taken. Weak chooses from a bounded, canonically ordered legal discard/non-winning-claim/pass list with seeded randomness; it may pass non-winning claims. Medium deterministically ranks legal discards by standard-hand grouping/readiness (meld/pair structure, then shanten; ties by tile/action order) and claims by resulting readiness. Strong starts with that ranking, examines a fixed top eight discard candidates with one-ply visible-tile improvement counts, then applies public discard/meld danger as a final tie-break; it uses no wall identities. Public remaining count is allowed. Crafted tactical fixtures, not chance alone, make Weak/Medium and Medium/Strong choose different valid actions.

Two full states differing only in hidden wall/opponent allocations, with the same observation, policy seed, and legal action set, must choose identically. Serialized API inspections and that behavioral comparison prove noninterference; compile-time types alone do not. No policy promises a win rate or needs a benchmark suite.

| Invariant | Evidence and requirement coverage |
|---|---|
| INV-1 phase-legal actions only; rejection has no partial mutation | Snapshot equality after invalid/stale action. R-4, R-5. |
| INV-2 tile conservation, deal/flowers/front wall/reserve lifecycle | Seeded flower chain, reserve draw, tile-count check. R-5, R-6. |
| INV-3 claims, kongs, priority, and progression are legal/deterministic | Three chow shapes, all kong paths, simultaneous-win fixture. R-5. |
| INV-4 decomposition, every active tai/exclusion, settlement and zero-sum totals match profile | Tai/exclusion fixtures, maximum decomposition, both payment formulas. R-5. |
| INV-5 private/public projections and AI decisions leak no hidden state | Serialized-view and paired-hidden-state tests. R-1. |
| INV-6 configured selectors apply to all three bots; policies materially differ | Default/changed selector fixture and crafted adjacent-policy fixtures. R-3. |
| INV-7 generated Wasm loads in the browser; after asset readiness gameplay makes no network calls | Static HTTP browser exercise plus network log. R-2. |
| INV-8 accessible human control and result path | Click, touch, keyboard, focus, labels, claim/pass, Next Hand/Restart exercise. R-4. |

R-1 maps to Purpose/INV-5; R-2 to paths/build/INV-7; R-3 to journey/AI/INV-6; R-4 to journey/INV-1/8; R-5 to rules/INV-1–4; R-6 to rules/INV-2. These are release gates.

## Slices, evidence, build, and gate

1. Verify availability, then pin an isolated Rust toolchain, Wasm target, matching `wasm-bindgen` crate/CLI, and lockfile. Start red with native tile/deal/flower/reserve/invalid-action tests.
2. Start red with claims, kongs, maximum-decomposition, every tai/exclusion, settlement, zero-sum, and dealer-progression fixtures; implement the profile only.
3. Start red with API serialization, paired hidden-state fairness, selector, and tactical AI fixtures; implement policies.
4. Start red with revision/restart and seeded browser win/wall-draw exercises; add Wasm, Worker, and accessible table.

Implementation first records the literal verified identifiers in `browser/rust-toolchain.toml` and `browser/README.md`; no global default changes. `rust-toolchain.toml` is metadata, not command routing. The recorded commands use `rustup run <verified-pinned-toolchain> cargo test --manifest-path browser/Cargo.toml` and `rustup run <verified-pinned-toolchain> cargo build --release --target wasm32-unknown-unknown --manifest-path browser/Cargo.toml`. Bootstrap uses `rustup toolchain install <verified-pinned-toolchain> --profile minimal`, `rustup target add wasm32-unknown-unknown --toolchain <verified-pinned-toolchain>`, then `rustup run <verified-pinned-toolchain> cargo install wasm-bindgen-cli --version <verified-matched-cli> --locked --root browser/.tools`. Generation uses `browser/.tools/bin/wasm-bindgen --target web --out-dir browser/web/pkg browser/target/wasm32-unknown-unknown/release/qkmj_browser.wasm`; serve with `python3 -m http.server --directory browser/web 8080`. The browser check opens that HTTP origin (web target needs HTTP, no bundler), confirms the generated Wasm loads, and records no post-load gameplay requests. Pinned values are deliberately verified and recorded during implementation, not claimed verified here.

Acceptance requires native seeded complete hands through win and reserve draw; flower replacement; all chow/pong/kong paths; claim priority; every active tai/exclusion; settlement/progression; and projection/action checks. The browser exercise covers the same seeded win/draw paths plus selector defaults/changes, bot progress, click/touch/keyboard/focus, legal claim/pass, Next Hand, Restart, real generated Wasm, and the post-load network check.

Risk is exhaustive legacy scoring; deterministic fixtures prove correctness while timing only proves responsiveness. This no-remote clone does not prove OS, credential, provider, or network isolation. Design Go must confirm the correction block, exact verified tool versions, and this rules profile before any source/config/dependency work. Rejected smaller alternatives: a draw/discard demo fails R-5; JS rules duplicate the testable Rust core; score simplification changes the contract; and an unisolated/global tool install is not reproducible.

Self-check: accepted requirements and corrected codewalk evidence frozen before specification.
