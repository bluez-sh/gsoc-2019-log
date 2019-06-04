# GSoC 2019 - FFmpeg - HEIF Support

### June 4: Tuesday

I guess I really underestimated the investigation time. It might take a few
more days to get a proper understanding of the format before I start working.
As of now, my aim is to modify the lavf/movenc.c (mov muxer) so as to produce
heif/heic files from it. This will help me get a better insight of the format
(as suggested by my mentor, Carl).

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
