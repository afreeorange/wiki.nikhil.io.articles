For when `youtube-dl` reports that you _can_ get a video in 1080p (or higher) but that you'll just get the video and no audio. You'll need `ffmpeg` for the merge.

```bash
youtube-dl \
  -f bestvideo[height=1080]+bestaudio \
  --merge-output-format mkv \
  --write-sub \
  https://www.youtube.com/watch?v=foobar
```

Note that you can use `>=` for the height.
