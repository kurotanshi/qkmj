* _2026-09-24 18:51:03 (gpt-5.6-luna/max)_
Implemented PTT-style viewport-limited 40em × 24em desktop scaling with readable mobile reflow, ANSI tile/rivers/melds layout, corrected compass mapping, result details, literal status output, and guarded keyboard/focus/lifecycle behavior.
Skipped live WASM/browser verification because the managed Chrome profile and local-port sandbox were unavailable; add it when the host browser is available.
Self-check: node --check app.js and the acceptance worker stub passed; .acceptance/check.mjs passed.
