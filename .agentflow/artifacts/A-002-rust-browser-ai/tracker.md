# Tracker

## Identity

- **Work key:** A-002-rust-browser-ai.
- **Active Ask:** A-002.
- **Goal:** Rebuild QKMJ in Rust/WebAssembly for browser play with selectable AI strength.
- **Last update:** 2026-09-24 17:11:19 Asia/Taipei.
- **Evidence commit:** 52fc2b88ffef6a930994f8975512c0539954d939.

## Overall state

- **State:** blocked.
- **Reason:** Reviewed design is ready; source work requires the owner's later Design Go for the exact plan commit.
- **Total:** 5.
- **Completed:** 1.
- **Remaining:** 4.

## Accepted task checklist

- [x] **T-1:** Establish browser and AI requirements from the owner's exact request, resolve local versus multiplayer scope, and map original game rules without modifying legacy C; proof is accepted requirements and current codewalk reports. Source: A-002. Proof: requirements-report.md, codewalk-report.md, and devlog RUN-003/RUN-004 host acceptance.
- [ ] **T-2:** Produce the smallest executable Rust/Wasm design with UI journey, exact rule coverage, AI policy differences, file scope and acceptance checks; freeze it in a plan commit and obtain required Design Go before product edits. Source: A-002.
- [ ] **T-3:** Implement the accepted Rust game engine and selectable AI with failing-first rule and legality tests, preserving legacy files; proof is focused and complete Rust tests against specified invariants. Source: A-002.
- [ ] **T-4:** Implement the accepted browser game interface and Wasm integration without a speculative service architecture; proof is a real browser journey from difficulty selection through a complete playable hand, restart, and error handling. Source: A-002.
- [ ] **T-5:** Independently review and verify the delivered checkout against every accepted requirement, document runnable commands and limitations, obtain required result gate, and commit/synchronize authorized changes; proof is exact-commit acceptance, relevant tests, browser evidence, and Git state. Source: A-002.

## Accepted scope changes

- None.

## Current recovery

- **Current item:** T-2.
- **Last proven result:** Design commit 52fc2b88ffef6a930994f8975512c0539954d939 passed independent plan-only review and host acceptance; product implementation remains unstarted.
- **Active blocker or running process:** Owner Design Go for 52fc2b88ffef6a930994f8975512c0539954d939; no worker is running.
- **Next safe action:** Receive Design Go for the reviewed plan, then proceed to Rust engine implementation and tests.
- **Expected changed files:** .agentflow/devlog.md; .agentflow/artifacts/A-002-rust-browser-ai/**. Product paths remain pending design approval.

## Completion proof

- **All accepted tasks checked:** no.
- **Blocking accepted decision:** Design Go: 52fc2b88ffef6a930994f8975512c0539954d939.
- **Operation running:** no.
- **Next action remaining:** T-2.
- **Evidence status:** current.
- **Judgment:** blocked.

## Update meaning

- Saving this tracker is a recovery checkpoint, not a stop signal.
- Work continues with the next unfinished item unless an independent stop condition applies.
