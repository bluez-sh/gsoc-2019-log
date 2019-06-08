# GSoC 2019 - FFmpeg - HEIF Support

### June 4: Tuesday

I guess I really underestimated the investigation time. It might take a few
more days to get a proper understanding of the format before I start working.
As of now, my aim is to modify the lavf/movenc.c (mov muxer) so as to produce
heif/heic files from it. This will help me get a better insight of the format
(as suggested by my mentor, Carl Eugen).

Today I mostly investigated the samples available in the original ticket.
MP4Box (by GPAC) can dump the "box" structure of heif files in a nicely detailed
xml format. With this and an hex editor, I tried to trace out the structure.
I also read through the code base a bit to understand how muxers work, I have
only worked on demuxers as of now.

Even though the iDevices use only a selected number of features or boxes, there
are still a lot of them, even for simple still images, let alone burst images,
live photos and what not. I had to lookup many boxes but I still didn't get all
of them as of now. Anyway, lets see if ISO/IEC 23008-12 helps fixing my sleep
cycle tonight!

### June 5: Wednesday

Surprisingly, it did not work. But on the up side, I did read quite a large
chunk of the specifications before I fell asleep! I guess I am quite clear on
the format, at least enough to keep me going for now.

Now my next step is to write a trivial heif muxer. So, today I spent most of
my time reading code. The mov muxer is a gigantic 7000 lines of code, tightly
packed, in a single file! At first it was quite daunting, naturally. But soon
I realised I don't have to worry about a large portion of the code. Figuring
out where to make the changes is quite challenging, but once you realise, you
feel stupid for not knowing it earlier. Of course the challenge doesn't end
there, I still have a long way to go, but I can see things unfolding and that
is a good sign.

Tomorrow I will try to make more sense of the code and hopefully start writing
the muxer.

### June 6: Thursday

One of the main features of heif is that it can support a wide range of codecs,
hevc being the focus as it provided the best compression (the reason it is known
for being twice as efficient as jpeg images in terms of space). Now it would
make sense if the muxer I write supports hevc by default. So I decided to go for
it, but the mov muxer itself doesn't support hevc, and I have a feeling there are 
a lot of changes to be made just for that. 

But I really don't know as of now, there are so many flags and macros lying
around I couldn't make any sense of. I don't even know how to ask the questions I
need to ask to get help. It is apparent I don't really understand the code yet,
but I am trying my best. I felt similarly when I wrote my first patch.

I did some minor changes for the muxer, and currently just experimenting with
the code to understand it better.

### June 7: Friday

I feel stupid (as I predicted). I have been worrying too much about the
things I shouldn't be worrying about, or at least not till much later. I guess
Carl could foresee this as well, he warned me ahead. A wise man, he is.

At last I wrote some meaningful code. Implemented some boxes. Can only test once
I am finished though.

I realised Apple uses 'tiles' extensively (another important feature of heif). 
It means sort of breaking a full image down into smaller images or 'tiles' (like
in a grid) and then processing them separately. This has a lot of advantages
including parallelism and efficient use of memory. But it is a pain to
implement. For now I'll have to make do with single image items.
