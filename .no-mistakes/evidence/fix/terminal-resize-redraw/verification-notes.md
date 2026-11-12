# Fresh terminal resize verification

The current CLI was built at b5faf3ced37a6db4b4e462f4c40372d558fadf16. The regression control builds that same CLI with only the renderer replaced by its blob at 3e1179f8460ac53779063d1ca9dfa2c02ec02e38. This is the only runtime source changed by the branch.

All three executions use the real CLI, renderer, orchestrator, synchronous Git commits, and a genuine PTY initialized to 120x40 before launch. A local synthetic Claude-protocol executable supplies successful iterations; no provider account, credentials, built-in mock mode, personal configuration, or network service is used. Each fixture has an isolated home, Git configuration, repository, cache and temp directory. Telemetry and sleep prevention are disabled.

The independent expected behavior is the user's delivered-notification contract: a visible title and complete prompt return on the first frame after signal dispatch, later ticks are differential, and one Ctrl+C finishes active work and restores terminal state. The baseline must remain damaged under the same blocked-hook sequence. The observers record real SIGWINCH and stdout resize events and forward stdout unchanged; they never inject an event or change product decisions.

The reused input harness ZIP SHA-256 is 0c6eb45786b5f96fdfd2fd44a9c2c47a2ad925cfe403c48c66ed0da2603fc382. Adaptations route evidence into this run, use the real agent loop for rapid/slow resizing, release the active synthetic iteration after graceful Ctrl+C, and compare read-only stty output at process exit. The first attempt's post-exit tcgetattr failed because macOS had revoked the PTY slave; its original records are retained in setup-attempt-1. Moving that observation into the existing exit observer fixed the setup, and every scenario was rerun successfully.

Commands run, from the worktree with Node 24.13.0 first on PATH:

- pnpm run build
- pnpm exec vitest run src/renderer.test.ts
- node .resize-verification/build-baseline.mjs
- python3 .resize-verification/drive.py
- node .resize-verification/replay.cjs
- swift -module-cache-path .resize-verification/swift-cache .resize-verification/render-images.swift

The supplied harness requires the existing xterm/headless 6.0.0 and Unicode 11 addon from the supplied external module directory. No runtime or emulator was downloaded. The latest successful driver, replay, build, tests and image-rendering commands all exited 0. The actual CLI processes each exited 130 after exactly one Ctrl+C, as expected. All recorded CLI/provider/hook processes had exited before cleanup. No source or permanent tests were changed, no broad test suite, linter, formatter or static analyzer ran, and no publication or pipeline-control operation was performed.

Scenarios and evidence:

1. Slow 80x24, 100x30 and 120x40 resizing: idle-candidate slow snapshots and results.json idle.slow.
2. Rapid 120x40 to 40x8 and back in approximately 54ms between frames: idle-candidate PTY/observations, rapid snapshots and results.json idle.rapid. Real signals were observed at both sizes; no terminal content was written while small. The next write was a complete frame containing the title and prompt.
3. Same round trip while the actual Git pre-commit hook blocks: git-candidate versus git-baseline PTY, observations and delayed snapshots. Neither received a stdout resize event; both received a delayed SIGWINCH at 120x40. Candidate restored title and prompt on its first frame after that signal (203ms); the baseline still lacked them.
4. Ordinary and subsequent animation: both candidate runs emitted differential writes without full repaints or prompt reprinting during the observed ordinary and post-recovery windows.
5. One Ctrl+C while the synthetic provider is active: graceful-wait and stopped snapshots, gnhf.log, end-state.json and Git log. Work completed and was committed; stdin raw mode, exact stty attributes, terminal title, cursor, normal screen buffer and original SIGWINCH listeners were restored. No provider survived.

The PNG rasterizer can clip or omit some box-drawing glyphs. The raw VT text and Unicode cell buffers retain the complete title; these images are labeled terminal replays, not desktop screenshots.

Windows was not run locally. The documented Windows missed-notification limitation is retained; this evidence makes no claim to fix it. README's concise scope and Refs #166 remain unchanged.
