1.  Reduce to a smaller size since
2.  "MPEG-1 does not support resolutions above 4095x4095"

for image in \*.JPG; do ffmpeg -i $image -vf scale=1200:-1
${image%.JPG}.PNG; done

1.  Create a movie!

ffmpeg -f image2 -pattern\_type glob -i "\*.PNG" output.mpg

1.  A lossless movie!

ffmpeg -pattern\_type glob -i "\*.PNG" -c:v mjpeg -qscale:v 0 output.mov

Resources
---------

-   [Scaling with
    ffmpeg](https://trac.ffmpeg.org/wiki/Scaling%20(resizing)%20with%20ffmpeg)
-   [Video from
    images](http://en.wikibooks.org/wiki/FFMPEG_An_Intermediate_Guide/image_sequence)
-   [Making a lossless
    movie](http://stackoverflow.com/questions/4839303/convert-image-sequence-to-lossless-movie)

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
