# ghdl — GitHub Download

Download files, videos, APKs, Docker images, and AUR packages into your repo using GitHub Actions — no local tools needed.

Each run saves files to a timestamped orphan branch (`dl/YYYY-MM-DD_HH-MM-SS`) with a `README.md` (direct links), `sha256sums.txt`, and `manifest.yml`.

---

## Setup

1. Fork / copy this repo (or drop the workflows into your own `.github/workflows/`)
2. **Settings → Actions → General → Workflow permissions → Read and write permissions → Save**

No secrets required for most workflows. See [optional secrets](#optional-secrets) below.

---

## Workflows

### ghdl — Generic URL downloader

**Trigger:** commit message *or* **Actions → ghdl → Run workflow**

Paste one or more `https://` URLs anywhere in a commit message, or enter them in the UI form.

```bash
# Single file
git commit -am "https://example.com/dataset.zip"

# MEGA file or folder
git commit -am "https://mega.nz/file/XXXXXXXX#YYYYYYYY"

# Media URL handled by yt-dlp (≤ 480p mp4)
git commit -am "https://youtu.be/dQw4w9WgXcQ"

# Multiple URLs in one commit
git commit -am "https://example.com/a.bin https://example.com/b.zip"
```

| URL pattern | Tool | Output |
|---|---|---|
| `mega.nz` file or folder | `megadl` | original filename(s) |
| Known video site / media extension | `yt-dlp` | `Title.mp4` |
| Any other `https?://` | `aria2c` (16 connections) | original filename |

---

### YouTube Downloader

**Trigger:** Actions → **YouTube Downloader** → Run workflow

Full-featured yt-dlp with quality, codec, subtitles, chapters, SponsorBlock, playlist range, and trim options.

| Option | Choices | Default |
|---|---|---|
| Format | mp4 / mp3 / best | mp4 |
| Quality | 4K / 1080p / 720p / 480p / 360p / 144p / best | 1080p |
| Codec | any / h264 / vp9 / av1 | any |
| Fragments | 1 / 4 / 8 / 16 | 4 |
| SponsorBlock | on/off | off |
| Subtitles | on/off + language | off |
| Trim | start/end timestamps | — |
| Playlist items | single / all / range / list | single |

Uses `bgutil-ytdlp-pot-provider` for automatic PO-token generation (no cookies needed for most videos). Store a Netscape cookie string as the `YT_COOKIES` repo secret for age-restricted content.

---

### Docker Image Downloader

**Trigger:** Actions → **Docker Image Downloader** → Run workflow

Enter an image name (e.g. `ubuntu:22.04` or `nginx:latest`). The image is pulled and saved as a `.tar.gz` archive.

**Load on your machine:** `docker load < docker_ubuntu__22.04.tar.gz`

---

### AUR Package Builder

**Trigger:** Actions → **AUR Package Builder** → Run workflow

Enter a package name (e.g. `yay` or `paru`). Built from source inside an `archlinux:latest` Docker container using `makepkg -s`. Outputs a `.pkg.tar.zst`.

**Install:** `pacman -U package.pkg.tar.zst`

---

### Google Play Downloader

**Trigger:** Actions → **Google Play Downloader** → Run workflow

Enter a package name (e.g. `com.google.android.youtube`). Downloads the APK for the chosen architecture (`arm64` or `armv7`).

**Requires repo secrets:** `GOOGLE_EMAIL` and `GOOGLE_PASSWORD` (Settings → Secrets and variables → Actions).  
A throwaway Google account is recommended.

---

### SoundCloud Downloader

**Trigger:** Actions → **SoundCloud Downloader** → Run workflow

Downloads tracks, playlists, or an artist's full discography. Embeds thumbnail and metadata automatically.

| Option | Choices | Default |
|---|---|---|
| Format | mp3 / flac | mp3 |
| Quality | 320k / 192k / 128k | 320k |

---

### Telegram Downloader

**Trigger:** Actions → **Telegram Downloader** → Run workflow

Paste one or more `t.me/…` links (space, comma, or newline separated — up to 1000). Downloads with resume support, retry logic, and 2 concurrent workers.

---

---

### Automatic branch cleanup

`dl/` branches are deleted automatically every **Monday at 03:00 UTC** if their last commit is older than **14 days**. You can also trigger it manually from **Actions → Cleanup old dl/ branches → Run workflow** and set a custom age.

---

## File splitting and branch chunking

**Step 1 — File splitting:** Any file over 90 MB is automatically split into 90 MB 7-Zip volumes so GitHub accepts them.

**Reassemble:** `7za x file.7z.001`

**Step 2 — Branch chunking:** Output files are grouped into batches of **15 files per branch** (~1.35 GB per branch). If a download produces more than 15 files, additional orphan branches are created automatically:

```
dl/2024-01-15_10-30-00            ← files 1–15  (parts 1–15 of a split archive)
dl/2024-01-15_10-30-00-part2of3   ← files 16–30
dl/2024-01-15_10-30-00-part3of3   ← files 31–45
```

Each branch has its own `README.md` with direct download links and a `sha256sums.txt`.

The `max_part_mb` input on each workflow lets you change the 7z split size (or set `0` to disable splitting).

---

## Optional secrets

| Secret | Used by | Purpose |
|---|---|---|
| `YT_COOKIES` | YouTube Downloader | Netscape-format cookies for age-restricted videos |
| `GOOGLE_EMAIL` | Google Play Downloader | Google account for Play Store auth |
| `GOOGLE_PASSWORD` | Google Play Downloader | Google account for Play Store auth |

---

## Notes

- All URLs must be publicly accessible (no login required) unless a cookie/auth secret is configured
- The workflow skips its own commits (`[skip ci]`) to prevent loops
- `dl/` branches are orphaned so they don't bloat your code history
- GitHub's free tier has a 6 GB repo limit; orphan branches keep your main history clean
