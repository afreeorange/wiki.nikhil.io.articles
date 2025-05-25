### Download the Best Quality Audio and Video

```bash
yt-dlp -f "bestvideo+bestaudio" URL
```

You can add `--merge-output-format mkv` for a container. Make sure `ffmpeg` is installed.

### Only Download Subtitles

If explicit subtitles are not supplied, will download auto-generated subs.

```bash
yt-dlp \
  --write-subs \
  --write-auto-subs \
  --sub-langs "en,es" \
  --sub-format "srt" \
  --skip-download \
  URL
```
