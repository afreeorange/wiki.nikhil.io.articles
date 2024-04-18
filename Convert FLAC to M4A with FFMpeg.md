This is for macOS. You'll need FFMpeg with [this encoder](https://github.com/mstorsjo/fdk-aac).

The 'regular' FFMPeg from HomeBrew will not work for [reasons](https://stackoverflow.com/a/55108365).

👉 Do _not_ install `fdk-aac-encoder` to fix this issue!

```bash
brew tap homebrew-ffmpeg/ffmpeg

# or 'upgrade'. The "--HEAD" will fetch the latest release.
# Might save you trouble with ImageMagick versions.
brew install homebrew-ffmpeg/ffmpeg/ffmpeg --with-fdk-aac --HEAD

# Convert!
find . -name '*.flac' -exec sh -c 'ffmpeg -i "$1" -c:a libfdk_aac -b:a 320k "${1%.flac}.m4a"' _ {} \;
```

## Problems

### ImageMagick Version

Just install the latest ImageMagick and FFmpeg with `--HEAD`.

### Could not find tag for codec h264 in stream #0 codec

FFmpeg is [trying to transcode any cover art](https://stackoverflow.com/a/52370948). To fix, just add a `-c:v copy` to the command above.
