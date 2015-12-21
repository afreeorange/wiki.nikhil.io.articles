```bash
# Reduce to a smaller size (1080p) since 
# "MPEG-1 does not support resolutions above 4095x4095"
for image in *.JPG; do ffmpeg -i $image -vf scale=1920:-1 ${image%.JPG}.PNG; done

# Create a movie!
ffmpeg -f image2 -pattern_type glob -i "*.PNG" output.mpg

# A lossless movie!
ffmpeg -pattern_type glob -i "*.PNG" -c:v mjpeg -qscale:v 0 output.mov
```

Resources
---------

- [Scaling with
  ffmpeg](https://trac.ffmpeg.org/wiki/Scaling%20(resizing)%20with%20ffmpeg)
- [Video from
  images](http://en.wikibooks.org/wiki/FFMPEG_An_Intermediate_Guide/image_sequence)
- [Making a lossless
  movie](http://stackoverflow.com/questions/4839303/convert-image-sequence-to-lossless-movie)


