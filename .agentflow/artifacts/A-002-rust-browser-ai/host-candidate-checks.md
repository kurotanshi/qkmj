# Candidate checks (not release acceptance)

Host reproduced the red stub compile failure, then imported only the 14 allowlisted candidate paths at the receipt digest; preserved AGENTS.md hash.

- Native cargo test: 14 passed, 0 failed. cargo fmt --check and node --check on both JS files passed.
- Bare `rustup run 1.98.1 cargo build --target wasm32-unknown-unknown` still invokes Homebrew rustc via PATH and cannot find core. Verified `env RUSTC="$(rustup which --toolchain 1.98.1 rustc)" RUSTDOC="$(rustup which --toolchain 1.98.1 rustdoc)" rustup run 1.98.1 cargo build --release --target wasm32-unknown-unknown --manifest-path browser/Cargo.toml` succeeds. Generated actual wasm-bindgen web package successfully.
- Real Chrome page http://127.0.0.1:8080/?seed=1 downloaded JS and Wasm HTTP200 but Start fails: Cannot convert 1 to a BigInt. Setup remains visually displayed despite hidden because CSS display overrides hidden.
- Independent native score probe: interior-pair outside fixture wrongly awards #35=4; closed-wait pinfu fixture wrongly awards #24=2; five-concealed fixture awards both #33=4 and #45=8; five-exposed one-tile fixture omits #23.
- This verifies the candidate is NOT complete despite baseline suite passing. Final worker result is still being prepared; imported candidate identity is frozen separately and will be compared before corrections.

## Attempt2 in-progress independent AI probe (not final result)
Host froze draft ai.rs into /tmp/qkmj-ai2-probe.rs and compiled with Homebrew rustc against live native dependency rlib (rustup and Homebrew builds are ABI-incompatible despite same reported semantic version). Real output:
- complete five melds + pair -> -3, expected -1;
- five melds + singleton -> -2, expected 0;
- five exposed + pair -> -1; five exposed + singleton -> 0 (accidentally correct due fixed clamp4).
Source uses fixed_melds.min(4), terminal8-2*meld_count, taatsu cap4. Must use full 5-meld shanten consistent with 16-tile game. Recheck final attempt2 before returning finding. Original labels remain 11..19筒 and21..29索 in draft engine tile_label, contrary original source.

## Attempt2 result-detail audit (in-progress)
Draft engine finish_win sets dealer_surcharge_tai when card_owner==dealer only. Original checkscr.c displays dealer surcharge for nondealer selfdraw OR dealer discard; dealer selfdraw already includes dealer/continuation in winning tai, so draft metadata is reversed for both selfdraw cases. Actual settlement() is separate: inspect final, preserve correct payments and fix only incorrect reported breakdown with regression if surviving.

## Follow-up at18:25 (recheck against final output)
Core draft corrected fixed_melds/terminal formula to5, but search_shanten still has `if taatsu < 4` at210. A 16-tile hand with FIVE partial groups plus pair [1,2,4,5,11,12,14,15,21,22,31,31,32,33,34,41] should have standard5-meld shanten4 (10-5-1), not5. Include this edge, not only already-complete hands.
Other pending draft checks: engine tile_label still11..19筒/21..29索 reversed; finish_win dealer_surcharge_tai wrong selfdraw metadata (seepriornote); Cargo.toml wasm-bindgen still nonexact "0.2.128" instead of "=0.2.128"; README cargo fmt fromroot lacks --manifest-path browser/Cargo.toml, and rustfmt/clippy component bootstrap is needed for those documented commands. Host installed clippy component1.98.1 at18:23. Positive all-claimed regression uses exposedpong31 pluspair31 (5 copies), should usepong34 oranotherdistinct legalmeld.
Owner newly provided original UI screenshot; independent reference-ui stage now owns app.js/index.html/styles.css. Preserve final core outputs, host will merge reference UI and compare lifecycle input. Do not fight concurrent UI work or edit live.

Host compiled latest frozen ai.rs and independently reproduced five-taatsu+pair =5, expected4; the four complete/tenpai probes now correctly give -1/0/-1/0.

## Host imported final attempt2
Final literal bundle hash db6b49fb3ff2a4c74b7bd078571a9cbf66f5ae1b4d39892a12f74bf0d235557c imported throughallowlist; new regression-first run failed on missing API/shanten. Corrected native suite passed15engine+13regression tests under explicit Rust1.98.1 compiler routing. Bounded remaining defects dispatched attempt3; no finalacceptance claimed. ReferenceUI frozeninput delta: final2 showError rerenderscontrols+focus, clears nextHandPending onlyupon successfulstate response instead of immediatelyafterrequest. Preserve these duringUImerge.
