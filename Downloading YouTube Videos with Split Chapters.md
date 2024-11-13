Using `yt-dlp`.

```bash
#!/bin/bash

eval YOUTUBE_URL=\${$#}

echo "Downloading \"$YOUTUBE_URL\""

if [[ -z "$YOUTUBE_URL" ]]; then
  echo "YouTube URL was not provided"
  exit
fi

DL_DIR_NAME_RAW=$(\
  yt-dlp \
  --print \
  --output="%(title)s" \
    "$YOUTUBE_URL" \
)

DL_DIR_NAME=$(echo "$DL_DIR_NAME_RAW" | sed 's|--output=||' | sed 's|.$||')

echo "Output will be written to \"$DL_DIR_NAME\""

mkdir -p "$DL_DIR_NAME"
cd "$DL_DIR_NAME"

yt-dlp \
  --format bestaudio \
  --audio-quality 0 \
  --extract-audio \
  --audio-format mp3 \
  --download-sections "*0:00-inf" \
  --split-chapters \
  --output="chapter:%(section_number)s %(section_title)s.%(ext)s" \
  --no-mtime \
  --no-playlist \
  "$@"
```

[Via](https://github.com/yt-dlp/yt-dlp/issues/3339#issuecomment-1873165594)

