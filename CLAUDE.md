# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**ghdl** is a GitHub Actions workflow that downloads files into a repo by parsing URLs from commit messages. There is no build system, no test suite, and no application code — the entire logic lives in `.github/workflows/ghdl.yml` (a single bash-heavy workflow file).

## How it works

1. Any push to any branch triggers the workflow.
2. The workflow parses the latest commit message for `docker://` or `http(s)://` URLs.
3. Each URL/reference is dispatched to the appropriate tool:
   - `docker://image:tag` → `docker save | gzip` → `docker_image__tag.tar.gz`
   - `aur://package-name` → Arch Linux container (`archlinux:latest`) + `makepkg -s` → `.pkg.tar.zst`
   - Known video/media URLs → `yt-dlp` (≤480p mp4)
   - All other `https?://` → `aria2c` (16 parallel connections)
4. Files > 90 MB are split with `zip -s 90m` (reassemble: `zip -s 0 file.zip --out full.zip && unzip full.zip`).
5. All output lands on a **new orphan branch** named `dl/YYYY-MM-DD_HH-MM-SS` with a `manifest.yml`.
6. The workflow tags its own commits with `[skip ci]` to prevent loops.

## Key files

- `.github/workflows/ghdl.yml` — the entire production implementation
- `resource/github-sandbox/` — an older, simpler predecessor (`download:`/`download-zip:` prefix syntax, commits files back to the source branch instead of an orphan branch); kept for reference
- `foo.txt` — scratch/test file

## Developing / testing

There is no local test harness. To test changes:
- Push a commit with a URL to a fork that has **Settings → Actions → General → Workflow permissions → Read and write permissions** enabled.
- Watch the Actions tab for the run; downloaded files appear on the resulting `dl/…` branch.

The workflow uses only standard Ubuntu tools (`aria2`, `zip`, `docker`) plus `yt-dlp` installed via `pip` and `deno` for YouTube PO-token generation — no custom actions or external secrets required.

## Commit message trigger syntax

```
# Generic file download
git commit -am "https://example.com/dataset.zip"

# Docker image
git commit -am "docker://ubuntu:22.04"

# AUR package (builds inside archlinux:latest container, outputs .pkg.tar.zst)
git commit -am "aur://yay"

# MEGA file/folder download
git commit -am "https://mega.nz/file/XXXXXXXX#YYYYYYYY"

# YouTube / media
git commit -am "https://youtu.be/dQw4w9WgXcQ"

# Multiple targets in one commit
git commit -am "docker://nginx:latest aur://paru https://example.com/model.bin"
```
