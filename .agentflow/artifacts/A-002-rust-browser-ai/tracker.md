# Tracker

## Identity

- **Work key:** A-002-rust-browser-ai.
- **Active Ask:** A-003.
- **Goal:** Rebuild QKMJ in Rust/WebAssembly for browser play with selectable AI strength.
- **Last update:** 2026-09-24 19:09:00 Asia/Taipei.
- **Evidence commit:** e3605a4b4394e0a5e2083d8de252eaf5cb4cee24.

## Overall state

- **State:** blocked.
- **Reason:** Implementation, browser verification and independent acceptance passed; exact Result Go is required by the invoked Agentflow skill.
- **Total:** 5.
- **Completed:** 4.
- **Remaining:** 1.

## Accepted task checklist

- [x] **T-1:** Establish browser and AI requirements from the owner's exact request, resolve local versus multiplayer scope, and map original game rules without modifying legacy C; proof is accepted requirements and current codewalk reports. Source: A-002. Proof: requirements-report.md, codewalk-report.md, and devlog RUN-003/RUN-004 host acceptance.
- [x] **T-2:** Produce the smallest executable Rust/Wasm design with UI journey, exact rule coverage, AI policy differences, file scope and acceptance checks; freeze it in a plan commit and obtain required Design Go before product edits. Source: A-002. Proof: design.md, cross-check-review.md, exact owner Design Go in A-003.
- [x] **T-3:** Implement the accepted Rust game engine and selectable AI with failing-first rule and legality tests, preserving legacy files; proof is focused and complete Rust tests against specified invariants. Source: A-002. Proof: host-attempt3-red.txt/native.txt/clippy.txt/wasm.txt, host-strong-hand.txt; 30 native tests and actual release Wasm generated.
- [x] **T-4:** Implement the accepted browser game interface and Wasm integration without a speculative service architecture; proof is a real browser journey from difficulty selection through a complete playable hand, restart, and error handling. Source: A-002. Proof: host-final-win.md, host-offline-draw.md, host-ui2-lifecycle.md, host-final-geometry.md/mobile.md and final-desktop.png.
- [ ] **T-5:** Independently review and verify the delivered checkout against every accepted requirement, document runnable commands and limitations, obtain required result gate, and commit/synchronize authorized changes; proof is exact-commit acceptance, relevant tests, browser evidence, and Git state. Source: A-002.

## Accepted scope changes

- **Change:** Recreate original Telnet/ncurses terminal layout and ANSI palette. Source: A-003 owner 「畫面請按照原版的訪 telent 進行設計.」 Effect: supersedes the approved felt-table visual styling; Rust/Wasm/rules/AI scope continues.

- **Change:** Owner supplied original game screenshot and requested closest possible reproduction. Source: A-003 「[Image #1] 介面請盡可能還原原版風格!」 Effect: supersedes approximate terminal dashboard; dense unboxed ANSI text tiles, real keyboard selection, original center result/right info/bottom messages. Earlier telnet-ui stage retained only as evidence, screenshot-driven reference-ui stage owns final web source.

- **Change:** Responsive terminal typography proportional to browser viewport. Source: A-003 「可以的話字體根據瀏覽器進行縮放?」「類似 https://term.ptt.cc/ 的風格」 Effect: supersedes fixed960px/11px layout; use shared scalable character unit, preserve original board proportions and usable touch controls.

- **Change:** Repair flower text clipping/overlap explicitly reported by owner screenshot. Source: A-003 「[Image #1] 花牌的文字被截半了」 Effect: confirms current finalUI3 visibility scope; include sideflower bounds and humanregion overlap checks atowner1251x819 viewport.

## Current recovery

- **Current item:** T-5.
- **Last proven result:** Final core3 native30/build/lint pass; UI2 lifecycle win/draw/next/restart/offline pass; frozen UI3 preview atowner1251x819 showsallfiveflowers/completehand,390phonehandy389 with44pxtargets.
- **Active blocker or running process:** None running; awaiting owner Result Go: e3605a4b4394e0a5e2083d8de252eaf5cb4cee24.
- **Next safe action:** Present tested and independently reviewed implementation for required exact Result Go; after receipt mark T5 complete.
- **Expected changed files:** .agentflow/devlog.md; .agentflow/artifacts/A-002-rust-browser-ai/**. browser/**, README.md, .gitignore are approved product paths.

## Completion proof

- **All accepted tasks checked:** no.
- **Blocking accepted decision:** Result Go: e3605a4b4394e0a5e2083d8de252eaf5cb4cee24.
- **Operation running:** no.
- **Next action remaining:** T-5.
- **Evidence status:** current.
- **Judgment:** blocked.

## Update meaning

- Saving this tracker is a recovery checkpoint, not a stop signal.
- Work continues with the next unfinished item unless an independent stop condition applies.
