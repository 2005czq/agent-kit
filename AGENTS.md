# Agent Guidance

## Principles
- Prefer repo commands, lockfiles, scripts, and conventions; keep edits focused and run the narrowest useful validation.
- No irreversible remote actions without explicit request. Always run `shellcheck` before executing a shell script.
- Follow XDG Base Directory conventions and Angular-style Conventional Commits (`type(scope): summary`).
- For `sudo` work, write and `shellcheck` a minimal `~/1.sh` (it is disposable) for the user to run.

## Environment & Tools

- WSL2 Ubuntu 24.04. Core tools include `wslu` package, `rg`/`rga`, `fd`, `jq`/`yq`, `gh`, `wrangler`, `hyperfine`, and `watchexec`.
- Documents: `pandoc`; `libreoffice`/`soffice --headless`; Poppler (`pdfinfo`, `pdftotext`, `pdftocairo`, `pdftoppm`, `pdfseparate`, `pdfunite`); `qpdf`, `mutool`, and `gs`.
- Media/OCR: `yt-dlp`, `ffmpeg`/`ffprobe`, `exiftool`, `magick`, `mineru-open-api` (prefer `extract --model vlm`), and `tesseract` (`chi_sim`, `chi_tra`, `eng`) as fallback.
- `whisper-ctranslate2` for STT, `edge-tts` for TTS, and `wa` for WolframAlpha calculations and factual queries.
- `~/agent-kit` is the source of truth for optional skills and MCPs. Follow `~/agent-kit/README.md` to manage them with `kst`. If a task needs a tool that is not installed, look there first, recommend a match, and ask before installing. Do not search GitHub for alternatives or install globally unless asked.

## Language Toolchains

- JS/TS: `fnm` + `pnpm`; Python: `uv`; Rust: `rustup` + `cargo`; Go: native `go`; C/C++: `clang` + `lld`, CMake + Ninja + `ccache`.
- Respect lockfiles and repo build commands; use `just` when a `justfile` exists.

## Files Deletion

- WSL paths: `trash-put -- <path>`. Windows paths (`/mnt/c`, `/mnt/d`, …): use the PowerShell Recycle Bin instead of `trash-put`.
- Use `rm` only when permanent delete is explicitly requested.
- Prefer working under WSL filesystem; touch `/mnt/*` only when necessary.

## Browser Automation

### Overall guidelines

- Before each action sequence, run `tab-list` and explicitly `tab-select` the task tab.
- Reuse the task's existing tab when its title/URL still matches; otherwise create one task tab. Add tabs only for workflows that genuinely require them.
- Sanitize `tab-list` output: keep only index, title, and base URL, stripping all extension URLs and query parameters.
- Separate actions from observations. Suppress routine `goto`/`click`/`fill`/tab-command stdout (keep stderr); always verify results with follow-up checks.
- Query via `find --regex` -> `snapshot --depth=3` -> `snapshot REF` -> full snapshot as a last resort. Refresh refs after DOM changes.
- Use `--raw eval | jq -c` for clean and necessary payloads, or save large output via `--filename` to query with `rg`/`jq`. Never dump raw HTML, full `body.innerText`, or large unfiltered collections.
- Use `run-code` with semantic waits (locators/states, no fixed sleeps) for multi-step tasks; return compact summaries.
- Use screenshots only for visual layout/canvas verification. Summarize console logs, network requests, and traces as artifacts.

### Chrome (Windows)

- Playwright CLI is configured to connect to the user's logged-in Windows Chrome through the extension. Before each Windows Chrome workflow, ask the user and wait for approval.
- Run `playwright-cli list` first; reuse `windows` if present, otherwise run `playwright-cli attach --extension=chrome` once.
- For inaccessible browser-internal pages (`chrome://`, `chrome-extension://`, ...) or profile data, use a scoped local source or ask the user.
- When work ends, close every non-deliverable tab in the Playwright group, including the welcome tab, then `detach`.

### Headless Chromium (WSL)

- For isolated Linux rendering and screenshots, use `playwright-headless -s=<unique-id>`, reuse that session name for subsequent commands, and finish with `close`.
