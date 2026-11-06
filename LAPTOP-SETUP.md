# Laptop setup

Changes on top of MPV Anime Build v5.6 for an HP Pavilion Gaming 15 (Ryzen 5 5600H, GTX 1650 4 GB, 8 GB RAM, 1366x768 60 Hz screen), Windows 11, mpv 0.41 (shinchiro).

## Settings: `mpv-laptop.conf`

Loaded after `mpv.conf` by `scripts/custom-config-loader.lua`, so it survives build updates.

- `vulkan-device=NVIDIA GeForce GTX 1650` and `d3d11-adapter=NVIDIA`: Vulkan lists the Radeon iGPU first, and mpv uses the first device, so the shaders ran on the iGPU.
- `video-sync=display-resample`, `interpolation=yes`, `tscale=oversample`: smoothmotion, removes 24 fps judder on a 60 Hz screen. `g` toggles it.
- `video-sync-max-video-change=1`: mpv's default. The build's 0.01 stops 29.97/59.94 fps video locking to 60 Hz.
- `ytdl-format`: streams up to 1080p and no AV1 (neither GPU decodes AV1).
- `[cache]`: 512 MiB ahead / 128 MiB back instead of 1 GiB + 1 GiB (8 GB RAM).
- `[Low-End]`: Power Saving also turns interpolation off.

## Fixes to build files

- `scripts/power_manager.lua`: checks the battery every 60 s instead of every 5 s (each PowerShell check costs about 3.5 s of CPU), and once when the first file loads.
- `scripts/skip_intro.lua`: follows `chapter-list` changes, so chapters added after loading (AniSkip) get the Skip button.
- `script-opts/youtube-download.conf`: `video_format=current` (`best` is a 360p pre-merged file on YouTube) and downloads go to `%USERPROFILE%\Videos`.

## Added

| Feature | Files | Keys |
|---|---|---|
| Torrent/magnet streaming ([mpv-gotorrent](https://github.com/BioHazard786/mpv-gotorrent) v1.0.4) | `scripts/mpv-gotorrent/`, `script-opts/gotorrent.conf` | `Alt+t` stats, `Alt+k` keep file (`p` and `Ctrl+k` are taken by the build) |
| Torrent status in the centre while connecting, starting and buffering | `scripts/gotorrent-status.lua` | |
| AniSkip: Opening/Ending chapters for releases without chapters, so the build's Skip button and timeline colours work | `scripts/aniskip-chapters.lua` | `Enter` skips (the build's key) |
| SponsorBlock for YouTube: skips sponsor, self-promo and subscribe reminders ([mpv_sponsorblock_minimal](https://codeberg.org/jouni/mpv_sponsorblock_minimal)) | `scripts/sponsorblock_minimal.lua`, `script-opts/sponsorblock_minimal.conf` | `b` toggles |
| Clip / GIF maker ([mpv-webm](https://github.com/ekisu/mpv-webm)), MP4 output to `Videos\Clips` | `scripts/webm.lua`, `script-opts/webm.conf` | `Shift+W` |
| Subtitle sync with [alass](https://github.com/kaegi/alass) ([autosubsync-mpv](https://github.com/joaquintorres/autosubsync-mpv) v0.33) | `scripts/autosubsync/`, `bin/alass-cli.exe`, `script-opts/autosubsync.conf` | `n` |
| yt-dlp location for mpv | `script-opts/ytdl_hook.conf` | |

Notes:

- `webm.conf` passes `--load-auto-profiles=no` to the encoder: the build's `Platform-Windows-Base` profile sets `vo=gpu-next`, which makes encoding fail.
- The mpv-gotorrent `install.ps1` picks the macOS binary on Windows (`darwin` contains `win`), so it was installed by hand from the release zip, checked against `checksums.txt`.
- `V` (NVIDIA VSR) needs an RTX 20-series or newer card; it does nothing on a GTX 1650.

## Paths to change on another PC

- `mpv-laptop.conf`: `vulkan-device` (list names with `mpv --vulkan-device=help`)
- `script-opts/ytdl_hook.conf`: `ytdl_path`
- `script-opts/autosubsync.conf`: `alass_path`
- `script-opts/webm.conf`: `output_directory`

## Outside the mpv folder

```powershell
winget install --id shinchiro.mpv -e
winget install --id yt-dlp.yt-dlp -e   # also installs FFmpeg and Deno

# Run mpv on the NVIDIA GPU (same as Settings > Display > Graphics > High performance)
$key = "HKCU:\Software\Microsoft\DirectX\UserGpuPreferences"
if (-not (Test-Path $key)) { New-Item -Path $key -Force | Out-Null }
New-ItemProperty -Path $key -Name "C:\Program Files\MPV Player\mpv.exe" -Value "GpuPreference=2;" -PropertyType String -Force

# mpv:// links, used by the mpv-gotorrent browser userscripts
$mpv = "C:\Program Files\MPV Player\mpv.exe"
New-Item -Path "HKCU:\Software\Classes\mpv\shell\open\command" -Force | Out-Null
Set-ItemProperty "HKCU:\Software\Classes\mpv" -Name "(Default)" -Value "URL:mpv Protocol"
Set-ItemProperty "HKCU:\Software\Classes\mpv" -Name "URL Protocol" -Value ""
Set-ItemProperty "HKCU:\Software\Classes\mpv\shell\open\command" -Name "(Default)" -Value "`"$mpv`" --force-window=immediate `"%1`""
```
