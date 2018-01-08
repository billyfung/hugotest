---
title: Videogrep
subtitle: Subtitle fun
date: '2015-10-09'
slug: videogrep
---
# Videogrep

So one of the things that I enjoy the most about Python is the vast number of
libraries that exist out there. (note to self, would be neat to have a random
python library finder) One of the recent libraries that I've come across is
[Videogrep][2], which is exactly what you think it is. It greps through
videos. You provide a video file, along with a subtitle file and then it will
search through subtitles for certain phrases, and apparently even patterns of
speech (I haven't personally tested this). And then it uses another awesome
Python library, MoviePy and mashes together clips from the movie where the
phrase/word is found. This produces quite amusing results if you have a movie
or videos where something is constantly repeated.

Here's somethign that I've made:

[![IMAGE ALT TEXT HERE][3]][4]

The repo itself requires some bit of work, so I might be spending some time
going through the code and fixing things up. I'm not sure it properly takes a
folder of videos to run through, I had to write a bash script to do it.

[2]: https://github.com/antiboredom/videogrep

[3]: http://img.youtube.com/vi/OhzLTDYmaqo/0.jpg

[4]: http://www.youtube.com/watch?feature=player_embedded&v=OhzLTDYmaqo