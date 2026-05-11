# ghdl — GitHub Download

Download anything into your repo just by writing a URL in a commit message.  
Files are stored in a separate `downloads` branch, organised by timestamp.

---

## Setup

1. Fork / copy this repo (or drop the workflow into your own `.github/workflows/`)
2. **Settings → Actions → General → Workflow permissions → Read and write permissions → Save**

No secrets or tokens needed.

---

## Usage

Put a URL or reference anywhere in your commit message:

```bash
# Download a file
git commit -am "https://example.com/dataset.zip"

# Pull a Docker image and save as tar.gz
git commit -am "docker://ubuntu:22.04"

# Build an AUR package
git commit -am "aur://yay"

# Download from mega.nz
git commit -am "https://mega.nz/file/XXXXXXXX#YYYYYYYY"

# Download a YouTube / media URL via yt-dlp
git commit -am "https://youtu.be/dQw4w9WgXcQ"

# Mix several targets in one commit
git commit -am "docker://nginx:latest aur://paru https://example.com/model.bin https://youtu.be/xyz"
```

You can also trigger it from the GitHub web UI: edit any file, write the URL as the commit message, and commit directly to `main`.

---

## How it works

| Prefix / pattern | Tool | Output |
|---|---|---|
| `docker://image:tag` | `docker save \| gzip` | `docker_image__tag.tar.gz` |
| `aur://package-name` | `makepkg -s` inside `archlinux:latest` | `package-version.pkg.tar.zst` |
| `mega.nz` file or folder URL | `megadl` | original filename(s) |
| Known video site / media extension | `yt-dlp` | `Title.mp4` |
| Any other `http(s)://` URL | `aria2c` (16 connections) | original filename |

**Large files** (> 90 MB) are automatically split into 90 MB 7-Zip volumes that GitHub will accept.  
To reassemble: `7za x file.7z.001`

All files land in a timestamped orphan branch (`dl/YYYY-MM-DD_HH-MM-SS`) along with a `manifest.yml` (source branch, commit SHA, file list) and a `sha256sums.txt` for verifying every file and part.

---

## Supported media sites (yt-dlp auto-detected)

YouTube, Vimeo, Dailymotion, Twitch, TikTok, Instagram, Facebook, Twitter/X,  
Reddit, Bilibili, VK, Rutube, Streamable, Odysee, Rumble, Kick, and any direct  
`.mp4 .mkv .webm .flv .avi .mov .m3u8` URL.

---

## Notes

- URLs must be publicly accessible (no login required)
- AUR packages are built from source inside an `archlinux:latest` Docker container; any build dependencies are installed automatically via `makepkg -s`
- Multiple targets in one commit are all processed in the same run
- The workflow skips its own commits (`[skip ci]`) to avoid loops
- GitHub's free tier has a 1 GB soft repo limit — the `dl/` branches are orphaned so they don't bloat your code history
