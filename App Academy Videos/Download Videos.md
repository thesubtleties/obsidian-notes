### Notice: 
All videos are publicly hosted on Vimeo/YouTube servers and are accessible without any login. This is just a collection of public video URLs - like making a playlist of videos that are already freely available online. No private content or credentials are involved in accessing these videos.

# Video Download Setup Guide

## 1. Install yt-dlp
```bash
python -m pip install -U yt-dlp
```

## 2. Install ffmpeg (Required for merging video/audio)

### Windows
Option 1 - Manual (Recommended):
1. Go to https://www.gyan.dev/ffmpeg/builds/
2. Download "ffmpeg-git-full.7z"
3. Extract the archive
4. Add the `bin` folder to your PATH or copy ffmpeg.exe to your yt-dlp folder

Option 2 - Using Chocolatey:
```bash
choco install ffmpeg
```

### macOS
Using Homebrew:
```bash
brew install ffmpeg
```

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install ffmpeg
```

## 3. Verify Installation
```bash
yt-dlp --version
ffmpeg -version
```

## 4. Basic Usage
Download from a list of URLs (one per line in urls.txt):
```bash
yt-dlp -a urls.txt
```

Common Options:
```bash
# Specific quality (e.g., 720p)
yt-dlp -f "bv[height<=720]+ba" -a urls.txt

# Best quality single file
yt-dlp -f "b" -a urls.txt

# Show available formats for a video
yt-dlp -F [URL]
```

## Troubleshooting
- If you get merge warnings, restart your terminal after installing ffmpeg
- If using Windows and added to PATH, you need to restart your terminal
- If ffmpeg isn't found, you can put ffmpeg.exe in the same folder as yt-dlp