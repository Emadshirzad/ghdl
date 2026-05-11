# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**ghdl** is a collection of GitHub Actions workflows that download files, videos, APKs, Docker images, and AUR packages into a repo. There is no build system, no test suite, and no application code — all logic lives in `.github/workflows/`.

## Workflow files

| File | Trigger | Purpose |
|---|---|---|
| `ghdl.yml` | push commit or `workflow_dispatch` (url input) | Generic `https://` downloads: MEGA, media via yt-dlp, or aria2c |
| `youtube.yml` | `workflow_dispatch` | Full-featured yt-dlp: quality, codec, subtitles, trim, playlist, SponsorBlock |
| `soundcloud.yml` | `workflow_dispatch` | SoundCloud track/playlist → mp3 or flac via yt-dlp |
| `docker.yml` | `workflow_dispatch` | `docker pull` → save as `.tar.gz` |
| `aur.yml` | `workflow_dispatch` | Build AUR package inside `archlinux:latest` → `.pkg.tar.zst` |
| `google-play.yml` | `workflow_dispatch` | Download APK from Google Play (needs `GOOGLE_EMAIL`/`GOOGLE_PASSWORD` secrets) |
| `telegram.yml` | `workflow_dispatch` | Download files from `t.me/…` links via telegramdownloader.net |
| `cleanup.yml` | schedule (Monday 03:00 UTC) or `workflow_dispatch` | Delete `dl/` branches older than 14 days |

## How it works

1. A download is triggered by either a push (commit message with URLs) or GitHub Actions UI ("Run workflow").
2. The appropriate tool handles each URL type:
   - `mega.nz` links → `megadl` (3 retries)
   - Known video/media URLs → `yt-dlp` (with bgutil PO-token for YouTube)
   - All other `https?://` → `aria2c` (16 parallel connections)
   - Docker → `docker save | gzip`
   - AUR → `makepkg -s` inside `archlinux:latest`
   - Google Play → `gplay-apk-downloader`
   - Telegram → Python downloader via `telegramdownloader.net` API
3. **File splitting:** Files > 90 MB are split into 90 MB 7-Zip volumes (`7za a -v90m`).  
   Reassemble with: `7za x file.7z.001`
4. **Branch chunking:** Output files are grouped into batches of **15 files per branch**.  
   If a download produces > 15 files (i.e., > ~1.35 GB), additional orphan branches are created:  
   `dl/YYYY-MM-DD_HH-MM-SS`, `dl/YYYY-MM-DD_HH-MM-SS-part2of3`, etc.
5. Each branch contains: downloaded files + `sha256sums.txt` + `README.md` (direct links) + `manifest.yml`.
6. Workflow commits carry `[skip ci]` to prevent trigger loops.

## Key files

- `.github/workflows/ghdl.yml` — generic URL downloader (push + workflow_dispatch)
- `.github/workflows/youtube.yml` — advanced YouTube/media downloader
- `.github/workflows/docker.yml` — Docker image downloader
- `.github/workflows/aur.yml` — AUR package builder
- `.github/workflows/google-play.yml` — Google Play APK downloader
- `.github/workflows/telegram.yml` — Telegram file downloader
- `resource/Meli-Action-main/` — reference implementation (older; used as source for youtube, google-play, telegram workflows)
- `foo.txt` — scratch/test file

## Developing / testing

No local test harness. To test:
- Push to a fork with **Settings → Actions → General → Workflow permissions → Read and write permissions** enabled.
- Or trigger via the Actions tab UI.
- Watch the Actions tab for the run; downloaded files appear on the resulting `dl/…` branch(es).

## Optional secrets

| Secret | Workflow | Purpose |
|---|---|---|
| `YT_COOKIES` | youtube.yml | Netscape-format cookies for age-restricted videos |
| `GOOGLE_EMAIL` | google-play.yml | Google account email for Play Store auth |
| `GOOGLE_PASSWORD` | google-play.yml | Google account password for Play Store auth |

## ghdl.yml trigger syntax (commit message)

```
# Generic file download
git commit -am "https://example.com/dataset.zip"

# MEGA file/folder
git commit -am "https://mega.nz/file/XXXXXXXX#YYYYYYYY"

# YouTube / media (≤480p mp4 via yt-dlp)
git commit -am "https://youtu.be/dQw4w9WgXcQ"

# Multiple URLs in one commit
git commit -am "https://example.com/a.bin https://example.com/b.zip"
```

For Docker, AUR, YouTube (with quality options), Google Play, and Telegram — use their dedicated `workflow_dispatch` workflows instead.
