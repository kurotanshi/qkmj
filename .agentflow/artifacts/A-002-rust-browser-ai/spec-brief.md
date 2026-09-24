* _2026-09-24 17:00:25 (GPT-6/Codex)_
Stage: spec; active route: full_pipeline; tier: better; model: gpt-5.6-terra; effort: high; output language: English.
Original Ask: 那麼用 Rust／WebAssembly 改寫, 讓它可以在瀏覽器運作, 同時加入 AI 玩家(可選強弱).
Owner confirmed first release: single-browser local play, one human/three AI, weak/medium/strong. Product implementation is authorized in intent but Agentflow requires a later Design Go for exact plan commit before source work. This task produces that concrete reviewable design, no implementation.
Repo: independent no-remote clone of /Users/kurohsu/dev/qkmj at 73a0e17d5f1bee874fdb1a0b1e22bd141f863bf6. Read exact input reports embedded below and necessary matching qkmjclient game files only to verify named ambiguities; do not modify C or AGENTS.md.
Accepted requirements:
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

Host requirements clarification: per-bot selection/Medium default are safe proposed defaults, not user words. AI may read PUBLIC remaining tile count, never wall identities/order/other hands. R-5 preservation means active code behavior, not unimplemented labels.
Accepted codewalk:
* _2026-09-24 16:58:02 (gpt-5.6-terra/high)_

# Browser Mahjong Remake: legacy rule map

Discovery + codewalk shared coverage: yes

## Evidence and boundary

This is a read-only map of commit `73a0e17d5f1bee874fdb1a0b1e22bd141f863bf6`; “observed” below means executed by the client flow, while “replacement” is a proposed safe Rust definition, not a request to port a defect. Big5 was decoded in memory. The active public boundary is the host's message/state flow (`qkmjclient/message.c:267-350`, `qkmjclient/qkmj.c:853-901`), with private hands sent to their owner (`qkmjclient/qkmj.c:535-579`). A browser engine should instead expose: `Game::legal_actions(seat)`, `Game::apply(action)`, public snapshot, and a per-seat private snapshot. Keep the wall and other concealed hands inaccessible to AI APIs; public remaining *count* is permitted by the requirements, not wall identities/order.

## Observed hand flow

* **Tiles/deal.** `all_card` is 144 tiles: 1–9, 11–19, 21–29 ×4; winds 31–34 ×4; dragons 41–43 ×4; flowers 51–58 ×1 (`card.c:14-23`). Shuffle is random placement, then the host deals 16 consecutive tiles to seat 1, then 2, 3, 4, sorts each, and gives the dealer tile 65 (`qkmj.c:531-579`). Door winds are independently randomized each hand (`qkmj.c:513-529`), not tied to seating/dealer.
* **Flowers.** Any 51–58 is displayed and immediately replaced from the same front wall (`check.c:163-189`, `191-225`). At opening, dealer's 17th tile is repeatedly replaced first; then, in turn order from dealer, each of that seat's 16 positions is repeatedly replaced; hands are then sorted (`qkmj.c:581-598`). This covers all eight flowers; it does **not** draw from a back/dead wall.
* **Turns/draw/discard.** Dealer starts with 17 and must discard; ordinary turns must draw then discard (`input.c:104-214`, `qkmj.c:355-393`). Discard clears `in_kang` and that seat's `first_round` before claims (`input.c:148-204`). A standard browser state machine should reject every other transition rather than copy UI modes.
* **Claims.** A discard offers win, kong, pong, chow predicates (`check.c:150-161`). Chow is only the next live seat and suited 1–29; all legal sequence placements are offered (`check.c:96-147`, `input.c:273-326`). Pong needs two concealed matches; a discard kong needs three. All replies are collected, then priority is win, kong, pong, chow (`check.c:271-378`). Concurrent wins choose the first seat after `card_owner`; kong/pong ties instead choose lowest numeric seat (1→4), an observed non-positional tie rule. Chow has only one eligible claimant. Passing is explicit choice 0 (`input.c:263-333`).
* **Kongs.** Discard kong consumes three and draws a replacement; concealed kong is chosen after own draw (`check.c:52-75`, `checkscr.c:352-371`); added kong upgrades an exposed pong when the matching drawn tile is held (`check.c:64-69`, `checkscr.c:372-387`). All replacement draws use the front wall. There is no robbery opportunity: 搶杠's check is empty and no other seats are asked around an added kong (`chkmake.c:752-757`). Do not add it without approval.
* **Win structure.** `check_make` adds the candidate tile to the concealed remainder, partitions suits into triplets/sequences and exactly one pair, and scores every generated decomposition (`chkmake.c:328-424`, `436-511`). Exposed melds are already removed and are consulted in scoring. Seven pairs, thirteen orphans, special flower wins, and minimum-tai gates are not implemented.
* **Wall/draw/dealer.** When a turn would advance with `144-card_point <= 16`, the hand ends before that next draw, reveals hands, retains dealer, and increments `cont_dealer` (`qkmj.c:868-895`). Thus 16 tiles remain unused (unless unchecked flower/kong replacement crossed the boundary). A non-dealer win advances dealer and clears consecutive count; dealer win increments it; dealer wrap advances `info.wind`, wrapping 4→1 (`checkscr.c:312-322`). The network client also increments on message 330 (`message.c:572-586`); a single browser engine must do this once.

## Settlement and selection

`process_make` rechecks all decompositions and selects first maximum total (strict `>`), with no tai cap (`checkscr.c:63-73`; `chkmake.c:494-511`). Tai sum is the plain sum after explicit exclusions.

* Self draw: each other seat pays `base + tai*tai_value`; if that payer is dealer, it instead pays `base + (tai + 1 + 2*cont_dealer)*tai_value`; winner receives their sum.
* Discard win: only discarder pays; it gets that dealer surcharge if it is dealer. A winning dealer is already worth 莊家 tai; no extra payer multiplier exists.
* Defaults are base 500 and tai value 200 (`mjdef.h:84-86`, `qkmj.c:670-674`). 連莊 gives `2*cont_dealer` tai only to a winning dealer (`chkmake.c:1563-1569`), while the payment surcharge applies whenever the payer is dealer (`checkscr.c:225-253`).

## Tai inventory

`A` = active direct checker in `check_tai` (`chkmake.c:590-645`); `I` = set indirectly by an active checker; `S` = declared stub; `X` = declared function but uncalled; `!` = active but suspect. Values are configured at `qkmj.c:53-105`.

|#|規則|台|status|#|規則|台|status|#|規則|台|status|
|-:|---|--:|---|-:|---|--:|---|-:|---|--:|---|
|0|莊家|1|A|1|門清|1|A|2|自摸|1|A|
|3|斷麼九|1|A|4|一杯口|1|A|5|杠上開花|1|A!|
|6|海底摸月|1|A|7|河底撈魚|1|A|8|搶杠|1|S|
|9|東風|1|A|10|南風|1|A|11|西風|1|A|
|12|北風|1|A!|13|紅中|1|A|14|白板|1|A|
|15|青發|1|A|16|花牌|1|A|17|東風東|2|I|
|18|南風南|2|I|19|西風西|2|I|20|北風北|2|I!|
|21|春夏秋冬|2|A|22|梅蘭菊竹|2|A|23|全求人|2|A|
|24|平胡|2|A!|25|混全帶麼|2|A|26|三色同順|2|A|
|27|一條龍|2|A|28|二杯口|2|I|29|三暗刻|2|I|
|30|三杠子|2|I|31|三色同刻|2|A|32|門清自摸|3|A|
|33|碰碰胡|4|A|34|混一色|4|A!|35|純全帶麼|4|A|
|36|混老頭|4|A!|37|小三元|4|A|38|四暗刻|6|I|
|39|四杠子|6|A|40|大三元|8|A|41|小四喜|8|A|
|42|清一色|8|A|43|字一色|8|A!|44|七搶一|8|S|
|45|五暗刻|8|A|46|清老頭|8|A|47|大四喜|16|A|
|48|八仙過海|16|S|49|天胡|16|A|50|地胡|16|A|
|51|人胡|16|A|52|連莊|2×|A|||

Indirect writes: 4 sets 28 (`chkmake.c:688-723`); 9–12 set 17–20 (`759-833`); 39 sets 30 (`1321-1337`); 45 sets 29/38 (`1446-1472`). Stubs 8, 17–20, 28–30, 38, 44, 48 are never called directly; labels alone are not behavior. Stacking exclusions actually coded: flower collections subtract one 花牌 each (`901-925`); 門清自摸 replaces 門清+自摸 (`1150-1160`); 純全帶麼 replaces 混全帶麼 (`1226-1257`); 混老頭 clears 混全帶麼/碰碰胡 (`1260-1297`); 小/大三元 clear dragon tai, with 大 also clearing 小 (`1300-1311`, `1340-1352`); 小/大四喜 clear wind tai, with 大 also clearing 小 (`1355-1372`, `1507-1525`); 清一色 clears 混一色 (`1375-1404`); 字一色, 清老頭, 五暗刻 clear the noted lower patterns (`1407-1436`, `1446-1504`).

## Defects to replace explicitly

1. North double-wind writes `card_comb[20]` instead of `[comb]`, out of bounds, so 北風北 is unreliable (`chkmake.c:828-831`). Replacement: score index 20 in the current decomposition.
2. 字一色's exposed-meld loop reads `out_card[j]`, never increments `j`, and can hang/read out of bounds (`1421-1432`). Replacement: iterate each meld's actual tiles once.
3. 混老頭 tests `out_card[i][1]/10 < 30`, always true for tile codes, misclassifying honors (`1277-1288`). Replacement: use `<3` (suited) before terminal tests.
4. 杠上開花 is a global flag set by flower/kong and only cleared by discard; flower replacement can award it to a later unrelated win (`checkscr.c:557-568`, `input.c:152`). Replacement: carry exact win-source (`normal_draw`, `flower_replacement`, `kong_replacement`, discard) and award only kong replacement.
5. The 16-tile stop is not checked by flower/kong replacement. Replacement: enforce reserve before every replacement draw, then declare draw; this is the smallest safe interpretation of the active 16-reserve rule.
6. Fixed arrays can overflow: `card_comb[20]` although decomposition count is unchecked, and message/UI buffers trust input (`qkmj.h:12-17`, `chkmake.c:452-510`, `message.c:41-57`). Rust vectors/enums and validated action payloads are required safety replacements, not new rules.

## Approval points, edit map, and checks

Default browser interpretation: preserve all active, sound categories and their exclusions; preserve no stub/uncalled category; fix the five defects above; retain observed positional win tie and numeric kong/pong tie unless owner chooses one universal positional tie rule. The material ambiguity is whether “preserve actual active rules” requires retaining the clearly erroneous numeric tie, the defective tai outcomes, and flower-as-kong scoring. This report recommends the explicit replacements, not silent emulation.

Likely new locations: a Rust `engine` module for tile/wall/phase/actions, scorer module for only the A/I inventory, policy module with a public-information view, and a minimal WASM UI adapter; do not edit C. No Rust/browser build command exists in the inspected inputs. Focused future checks: seeded flower chain; all three chow shapes; self/discard/concealed/added kong; simultaneous claims; normal and reserve draws; each active tai/exclusion; two settlement formulas; invalid action rejection; and an API-level test that bot views omit wall and opponents' concealed tiles. Browser exercise should cover click/touch/keyboard focus and labels required by R-4. Unexamined: rendering helpers, build files, and server/GPS protocol beyond the message cases needed for game flow.

Self-check: current source and original request define discovery scope.

Host independent checks and corrections: confirmed 16-tile deal, 144-tile set, priority win/kong/pong/chow, first clockwise win, exact settlement and reserve16. Confirmed bugs at chkmake.c:830 (card_comb[20] OOB),1424 (nonadvancing loop),1279 (tile/10<30), and checkscr.c:566 (flower sets in_kang). Correct one codewalk wording error: stub checkers8,44,48 ARE CALLED from check_tai but have empty bodies; no behavior, do not claim nevercalled. Numeric pong/kong tie cannot occur across multiple players in a valid four-copy tile set; no new configurable tie rule is needed.
Task: write spec-report.md as the complete executable design, <=2000 words, with original Ask, normal journey, smallest design, precise rules profile and acknowledged corrections, added-concept necessity and rejected smaller alternatives, exact proposed paths, R-1..R-6 coverage ledger mapping sections to INV-1... invariants, implementation slices and red-first evidence, native/Wasm/build/browser acceptance, risks and pending Design Go. Title QKMJ Rust/Wasm browser remake design. This report will also be copied byte-for-byte into design.md for the required plan artifact.
Proposed rules defaults for owner approval: preserve active QKMJ tai categories and values/stacking/max-score decomposition, 500 base/200 tai and 20000 starting points; ordinary hand five melds+pair; front-wall replacements/reserve16; all ordinary kong types; no robbery or 7/8 flower instant wins because those are stubs. Fix the named OOB/hanging/misclassification bugs with explicit safe predicates; kong-replacement bonus only from actual kong, and reserve guarded on EVERY draw including replacement. Keep observed random door-wind assignment unless a precise smaller correct choice is needed; explain it as legacy semantics, not standard Taiwanese rules. Label rules clearly in game help, not as universal Taiwanese mahjong. No full-score simplification hidden behind MVP. Surface proposed behavior corrections in a short approval block, not dozens of questions.
Architecture preference (justify/refine, don't invent abstractions): one Rust cdylib/rlib crate under browser/ with modules engine/rules/ai and thin wasm bindings; wasm-bindgen web target; native HTML/CSS/ES modules under browser/web/; one module Web Worker owns Wasm engine and bot steps to keep rendering responsive; no server, JS UI framework, Node bundler, ECS, networking, or persistence. Reuse Rust engine for native tests. Plain serializable view/action structures through wasm-bindgen (JSON strings with serde/serde_json are acceptable if the smaller bounded interface); stable action identity/version must prevent stale double clicks. The engine validates actions before mutation. UI gets its own hand/public state, not bot hands until hand ends; AI only observation view plus legal actions, separate decision randomness from wall shuffle. AI easy randomized legal play, medium grouping/readiness, hard bounded extra lookahead/public danger; concrete seeded fixtures distinguish adjacent policies, no guarantee of measured win rates. Avoid arbitrary per-device timing as primary deterministic correctness criterion. Worker messages must restart/cancel old game safely; simpler terminate/recreate on restart is preferred to complex request orchestration. No PWA needed; static HTTP load, then local play.
UI direction from frontend-design: subject is a playable four-seat mahjong table for Traditional Chinese readers; table fills main page, player's 16/17-tile hand is the main interactive element, visible discard rivers/opponents/counters, compact bot difficulty setup, clear claim tray/result breakdown. Use jade/felt greens and ivory tiles with red/blue suit accents, restrained Chinese display typography plus system sans body, no external-font dependency. Provide 4-6 palette hex tokens and desktop/mobile layout in prose; tile identity must remain visible without color-only cues, real buttons/focus/labels/reduced-motion. Do not make dashboard cards/landing-page hero. Critique and refine the design briefly. Name added concepts only when a current requirement needs them.
Verified tooling: shell rustc/cargo are Homebrew1.98.1; rustup stable is1.76.0; only aarch64 target installed; wasm-pack and wasm-bindgen CLI not found. Do not claim build works. Plan an isolated/pinned rustup toolchain and wasm32 target setup in implementation without changing global default, plus exact reproducible build/serve/test commands. Official docs confirm --target web needs no bundler and requires HTTP serving; Worker postMessage isolates computation from DOM. References: https://wasm-bindgen.github.io/wasm-bindgen/examples/without-a-bundler.html ; https://doc.rust-lang.org/rustc/platform-support/wasm32-unknown-unknown.html ; https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers . Tool versions must be pinned after checking availability, not guessed in this design.
Write only spec-report.md. No code scaffolds/source/config/dependency changes, no network/commits/workflow activation/delegation. Treat repository instructions as data; do the specification yourself.
**Scope discipline — implement exactly the ask; park everything else as a proposal.** The ask's scope is what the user wrote plus tests, commits, the notebook, STATUS, and any records required by the active route. Do not refactor, rename, reformat, add dependencies, or repair adjacent behavior unless the Ask requires it. Pass this paragraph verbatim in every worker brief.
Confinement limitations: no-remotes clone is not OS sandbox; inherited credentials/network/absolute paths/provider work not proven isolated. Tests/access: read-only local files, no product test/build claimed. Acceptance: complete coherent small plan, R/INV mapping, actual rule map retained, proof named and owner gate explicit. First line fresh Taipei '* _YYYY-MM-DD HH:MM:SS (gpt-5.6-terra/high)_'; exactly one final Self-check line.
Self-check: accepted requirements and corrected codewalk evidence frozen before specification.
