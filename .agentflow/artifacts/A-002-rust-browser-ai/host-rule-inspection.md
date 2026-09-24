# Host inspection checkpoints

Directly inspected Big5-decoded legacy source without modifying it:

- `qkmj.c:513-518`: scoring winds are a randomly rotated 1–4 order, not an arbitrary permutation or four independent choices.
- `qkmj.c:535-598`: consecutive 16-tile packets per physical seat, sorted before opening flowers; dealer extra tile is replaced first, then initial sixteen positions in dealer order.
- `chkmake.c:309-311`: decomposition traversal is triplet, sequence, pair. Maximum score selection retains first equal maximum.
- `chkmake.c:657-723,1150-1160`: legacy closed-hand tests use concealed count (`num==16`), so even concealed kong changes this condition; do not silently apply modern closed-kong semantics.
- `chkmake.c:989-1026,1030-1089`: mixed terminal/honor groups, three-color sequences, and straight lose one tai when not legacy-closed. Pure terminal groups do not have this reduction.
- `chkmake.c:936-986`: pinfu requires no flowers, all sequences, suited non-winning pair, discard win, and another valid winning tile at +/-3; use structural predicate without recursive scoring.
- `chkmake.c:862-925`: matching scoring-wind flowers award one each; complete season/gentleman collection each awards two and removes its one matching-flower tai.
- `chkmake.c:1446-1472`: discard-winning triplet matching winning tile is not concealed; concealed kong counts. Three/four/five concealed sets are mutually exclusive; five clears all-triplets.
- `chkmake.c:1534-1559`: first-round flags are per-player before their first discard, not a global turn number.
- Correct only the design-approved North-double-wind index, honor iteration, exposed mixed-terminal predicate, flower-versus-kong win source, reserve checks, and memory/input safety.

These are host verification checkpoints, not new requirements or implementation evidence.

## Draft implementation checkpoints (not yet an accepted result)

First rules.rs inspection while worker active identified items to recheck after delivery: pair-first enumeration versus legacy triplet/sequence/pair; 全求人 exposed count must permit five melds with a one-tile remainder; 平胡 must check alternate +/-3 structural wait; outside-hand patterns must include the pair and legacy concealed-side existence condition; 五暗刻 must clear 碰碰胡 after its award. These affect the approved complete legacy-rule behavior and therefore are in scope for correction if still present in delivered result.

Draft engine/AI inspection adds these final-delivery checks: all discard claims including Win/DiscardKong must go through reply collection before resolution; bot_step must not make human choices and must stop for a meaningful human claim; opening actual extra tile must survive sorting as winning tile; scoring winds must rotate rather than arbitrary shuffle; round result needs winning tile and decomposition; dragon 42 is 白板 and 43 is 青發. AI draft readiness double-counts overlapping groups and lacks approved standard shanten, ignores Pass when any claim exists, counts claimed river tiles again in visible availability, and own concealed tiles should not be mistaken for public danger. Confirm the delivered version fixes these or return them with regression evidence to implementation.

Native fixture candidates for direct source-derived expectations:
- Interior-pair outside false: completed [1,2,3,7,8,9,11,12,13,17,18,19,21,22,23,25,25] must not receive #25/#35 despite every meld containing a terminal.
- Closed-wait pinfu false: completed [1,2,3,11,12,13,21,22,23,24,25,26,27,28,29,5,5], discard winning 2; substituting 5 does not structurally win, so #24 is absent.
- Full exposure: five exposed sequences plus concealed [31], discard-winning 31 yields #23=2.
- Five concealed triplets: [2,2,2,5,5,5,12,12,12,16,16,16,25,25,25,31,31], self draw gives #45=8 and #33=0.
- Wasm action_json currently accepts arbitrary actor in draft: enforce human actor0 plus bounded malformed input before mutation, tested at native boundary/shared parser and realWasm.
- tile_conservation should compare per-code counts to the 144-tile multiset, not merely total length.

Draft UI/Wasm checks: Rust u64 constructor requires JS BigInt, not Number. Phase is externally tagged/string in draft while app tests phase.type, so result must explicitly stop pumping bots. Difficulty controls must reappear between hands (currently setup permanently hidden); restart must return to functional setup/recovery. Include result decomposition, dealer continuation and claimant/source, not just totaltai. Correct 42/43 labels throughout UI and Rust. Preserve meaningful focus when a tile disappears; do not render a second tray of all discard buttons. Worker game/error state must not leave the user stuck with hidden recovery controls.

Hidden-state fairness boundary to test: Observation currently embeds PublicState.needs_human, computed from human claim availability; changing the human concealed hand can therefore change a bot observation even with identical true public state. UI-private action-availability fields must not enter AI observation. Paired hidden-state test must vary both wall order and opponent allocations while preserving legal actions and public counts, not just replace a synthetic wall.

The later draft's `direct_tai_flags_and_round_boundaries_are_regressed` all_claimed fixture uses concealed [1,2,3,4] + winning4 with only FOUR exposed melds and expects #23. This expectation contradicts chkmake.c:932 (`num==1`); correct the test and implementation, not legacy semantics. Add the negative four-meld case as well as the positive five-meld case. Draft setup browser inspection also exposes implementation copy (“Rust/WebAssembly”, “自然亂數種子”) in the main player flow; keep reproducibility in optional ?seed= query/help and use normal randomized sessions by default.
