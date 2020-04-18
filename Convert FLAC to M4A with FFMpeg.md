This is for macOS. You'll need FFMpeg with [this encoder](https://github.com/mstorsjo/fdk-aac).

```bash
brew tap homebrew-ffmpeg/ffmpeg

# or 'upgrade'
brew install homebrew-ffmpeg/ffmpeg/ffmpeg --with-fdk-aac

# Convert!
find . -name '*.flac' -exec sh -c 'ffmpeg -i "$1" -c:a libfdk_aac -b:a 320k "${1%.flac}.m4a"' _ {} \;
```
