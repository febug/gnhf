# Terminal resize verification

Verdict: go for the scoped macOS delivered-notification recovery at `adda1ad6c9486c003057d33ab7f2cdd7266eab6f`.

The current bundled CLI was run in a real macOS PTY initialized to 120 x 40 before launch. The fixture used the synthetic prompt `ship a readable terminal after resize`, isolated home/config/cache/tmp and Git repository, telemetry disabled, and a synthetic external Claude-protocol executable. The CLI, renderer, agent loop, synchronous Git commits, and blocking pre-commit hook were real. No provider account or credentials were used. The built-in offline demo was supplemental, not the sole evidence for any scenario.

## Scenarios and independent oracle

The expected behaviors come from the supplied user intent: a delivered notification restores title and prompt on the next frame, ordinary frames remain differential, and one Ctrl+C completes active work and restores the terminal. Captured PTY bytes were interpreted with xterm/headless 6.0.0 and Unicode 11; title and prompt are checked in the resulting terminal cells. A preload observer records actual OS signal dispatch, stdout resize events, frame writes and listener identity without injecting events or replacing product output.

| End-user action | Observed result |
| --- | --- |
| Resize the real CLI slowly to 80 x 24, 100 x 30, then 120 x 40 | Title and full synthetic prompt readable at each size. |
| Shrink 120 x 40 to 40 x 8 and return about 50 ms later between frames | No output while small. The terminal loses content, then the first repaint restores title and prompt. |
| Perform that round trip while a real synchronous Git hook blocks dispatch | No output while blocked. SIGWINCH arrives at the original 120 x 40 dimensions; stdout emits zero resize events. First frame after actual signal delivery is full and restores both title and prompt, 203 ms after delivery. |
| Keep watching after recovery | Ordinary observation windows and both post-recovery windows contain differential animation writes with no full repaint. |
| Press Ctrl+C once during the second synthetic iteration | Graceful-wait hint appears, the active iteration completes, both changes are committed, the exit summary reports two successful iterations, normal terminal buffer and cursor return, raw input is off and the added listener is removed. The supplemental offline run also verifies exact termios restoration and no output after a post-stop resize. |

The renderer from base commit `3e1179f8460ac53779063d1ca9dfa2c02ec02e38`, bundled with otherwise identical runtime source, reproduces missing title and prompt for both rapid-resize sequences. Its first frame after delayed SIGWINCH is differential and remains unreadable. This is the expected negative control, not a failure of the candidate.

## Evidence

- [Rendered terminal replay](terminal-replay.html)
- [Rapid resize before recovery](git-candidate-rapid-before-repaint.png) and [after recovery](git-candidate-rapid-recovered.png)
- [Candidate after blocking Git hook](git-candidate-delayed-recovered.png) and [base renderer after the same sequence](git-baseline-delayed-recovered.png)
- [Slow 80 x 24 resize](git-candidate-slow-80x24.png)
- [Graceful-wait display](git-candidate-graceful-wait.png), [permanent exit summary](git-candidate-stopped.png), [persisted run state](git-candidate-end-state.json) and [real Git log](git-candidate-git-log.txt)
- [Raw PTY, VT bytes, actual signal/write observations and terminal cells](terminal-captures.zip)
- [Detailed measured results](results.json), [reusable adapted harness](verification-harness.zip), [build provenance](provenance.json)

These images are terminal replays of this run's captured output, not desktop screenshots. The PNG rasterizer clips or omits some box-drawing strokes; the original VT bytes and cell text retain the complete title. The image limitation does not affect the prompt, signal/frame oracle or stored terminal cells. No browser was used.

## Commands and setup

Node 24.13.0 and pnpm 11.1.1 were used with the already-installed dependencies.

- `pnpm run build`
- `node .resize-validation/harness/build-baseline.mjs`
- `pnpm exec vitest run src/renderer.test.ts -t 'Renderer resizing|Renderer ctrl.c|Renderer terminal title'`
- `python3 .resize-validation/harness/drive.py`
- `python3 .resize-validation/harness/drive.py git-baseline`
- `node .resize-validation/harness/replay.cjs`
- `swift -module-cache-path .resize-validation/swift-cache .resize-validation/harness/render-images.swift`
- Visual inspection of the generated rapid, slow, blocked-hook and shutdown replays; persisted state and Git-log inspection; scoped process-exit and cleanup checks.

The initial adapted baseline bundle was placed one directory too deep, so it exited before TUI startup while loading `../package.json`. The output location was corrected to `.resize-validation/cli.mjs`, rebuilt, and the baseline was run successfully. The failed setup capture is retained under [setup-attempt](setup-attempt/README.txt). A later inspection requested a non-existent combined iteration file; the actual per-iteration JSONL and `end-state.json` were then retained. Neither issue required a source change or weakening an assertion.

The candidate build, corrected baseline build, targeted tests, PTY comparisons, replay assertions and rasterizer all completed. CLI interrupt exit code 130 is expected; the demo exited 0. No full test suite, lint, formatter or static analyzer was run. Earlier complete-suite results from the supplied plan were not treated as current results.

## Scope and cleanup

Windows was not run locally. The accepted pre-existing case where Windows produces no notification remains outside this fix. The existing README retains that limit and the non-closing `Refs #166` reference. No source, tests, documentation or commit history were changed in this phase.

All spawned CLI, synthetic-provider, Git-hook, build, test, replay and image-rendering children finished. Task-only bundles, harness working files, fixture repositories and Swift cache were removed; prepared dependencies were preserved. Evidence remains in this run's dedicated evidence directory.
