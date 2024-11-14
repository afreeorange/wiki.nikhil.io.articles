Using `yt-dlp`.

```bash
#!/bin/bash

eval YOUTUBE_URL=\${$#}
if [[ -z "$YOUTUBE_URL" ]]; then
    echo "YouTube URL was not provided"
    exit
fi

DOWNLOAD_DIR_NAME_RAW=$(
    yt-dlp \
        --print \
        --output="%(title)s" \
        "$YOUTUBE_URL"
)

DOWNLOAD_DIR=$(echo "$DOWNLOAD_DIR_NAME_RAW" | sed 's|--output=||' | sed 's|.$||')

echo "Output will be written to $DOWNLOAD_DIR"

mkdir -p "$DOWNLOAD_DIR"
cd "$DOWNLOAD_DIR" || exit

# For just Audio, use these flags:
#
#  --format bestaudio \
#  --audio-quality 0 \
#  --extract-audio \
#  --audio-format mp3 \

yt-dlp \
    --download-sections "*0:00-inf" \
    --split-chapters \
    --output="chapter:%(section_number)s %(section_title)s.%(ext)s" \
    --no-mtime \
    --no-playlist \
    "$@"
```

[Via](https://github.com/yt-dlp/yt-dlp/issues/3339#issuecomment-1873165594)

### A Slightly Faster Way

First this

```bash
yt-dlp --write-info-json -o "%(title)s.%(ext)s" <video_url>
```

Then, assuming the downloads are called `video.json` and `video.webm`,

```javascript
const fs = require("node:fs");
const path = require("node:path");
const { exec, execSync } = require("node:child_process");

const videoFile = "video.webm";
const jsonFile = "video.json";

/**
 * Left Pad
 *
 * https://github.com/left-pad/left-pad/blob/master/index.js
 */
function leftPad(str, len, ch) {
  str = str + "";
  len = len - str.length;
  if (len <= 0) return str;
  if (!ch && ch !== 0) ch = " ";
  ch = ch + "";
  if (ch === " " && len < 10)
    return (
      [
        "",
        " ",
        "  ",
        "   ",
        "    ",
        "     ",
        "      ",
        "       ",
        "        ",
        "         ",
      ][len] + str
    );
  var pad = "";
  while (true) {
    if (len & 1) pad += ch;
    len >>= 1;
    if (len) ch += ch;
    else break;
  }
  return pad + str;
}

// Load the JSON metadata
fs.readFile(jsonFile, "utf8", (err, data) => {
  if (err) {
    console.error("Error reading JSON file:", err);
    return;
  }

  const metadata = JSON.parse(data);
  const chapters = metadata.chapters || [];

  // Create a directory to store the split chapters
  const outputDir = "chapters";
  if (!fs.existsSync(outputDir)) {
    fs.mkdirSync(outputDir);
  }

  /**
   * Split the video into chapters using ffmpeg. Use the H.264 and AAC
   * codecs.
   */
  let counter = 1;
  for (const chapter of chapters) {
    const startTime = chapter.start_time;
    const endTime = chapter.end_time;
    const title = chapter.title;
    const sanitizedTitle = title.replace(/[^a-z0-9 _-]/gi, "_");
    const outputFile = path.join(
      outputDir,
      `${leftPad(counter, 2, "0")} - ${sanitizedTitle}.mp4`
    );

    const ffmpegCommand = `ffmpeg -i "${videoFile}" -ss ${startTime} -to ${endTime} -vcodec h264 -acodec aac "${outputFile}"`;
    execSync(ffmpegCommand, (error, stdout, stderr) => {
      console.log(`Starting chapter "${title}"...`);

      if (error) {
        console.error(`Error splitting Chapter "${title}":`, error);
        return;
      }

      console.log(`Chapter "${title}" has been saved to "${outputFile}"`);
    });

    counter += 1;
  }
});
```
