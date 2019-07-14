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

### June 8: Saturday

Implemented all remaining boxes which are essential, only mdat remains. Tomorrow
I'll take care of mdat, and finally test the muxer, and hope it works (which it
most probably won't).

I might have to read some more code to find out how exactly the image data
received from a jpeg file will be written into mdat.

### June 10: Monday

Due to circumstances I had to take a day off, I hope it is allowed. Anyway I
guess taking a day off at weekend is okay sometimes. I plan to make up for it 
in the weekdays though.  

Finally finished writing the muxer, tested it, and it worked! It converted a
jpeg file into heic without any problems! And I was so happy it worked right
away! And as I inspected it with MP4Box, I found the hevc config (hvcC tag) was
not written properly, dunno why. So... no heic for me yet. :(

I used the write_hvcc_tag function which was already defined in the mov muxer,
maybe there is something different with the way it is implemented in heic, I
need to check it out properly. I'll try to fix it now, maybe ask in the irc (or
Carl if he is available), or I'll do it tomorrow.

### June 11: Tuesday

It worked! I fixed it by morning. Apparently memory wasn't allocated for
extradata that was needed to write hevc decoder config. I just tried adding that
code before calling in heic related functions, and it worked like a charm!

So I tested it on a few jpeg files, and the results were pretty astonishing. A
2.8M jpeg file was converted into a heic file of 350K! Now that's way less than
50% the original size, and that too with practically no visible difference! Now
that's a lot of exclamations but you get the point, hevc compression is indeed
"state-of-the-art" as Apple likes to call it.

I talked to Carl and he suggested I start working on the heic demuxer and make
it read the files I generated using the muxer. Pretty neat. I guess I'll start
tomorrow, or maybe tonight.

### June 12: Wednesday

I spent most of my time reading the mov demuxer code. Aside from that, I found
the heic images that my muxer produces doesn't work on OSX for some reason, says
they're broken. When I uploaded to google drive I could view it though. I
couldn't yet identify why is it so, but I'll find it soon enough. Maybe its
because I didn't implement a few boxes, which didn't seem much relevant at that
point.

I plan to start writing a basic demuxer today (Thursday as of writing).

### June 13: Thursday

I spent a good amount of time in understanding clearly how the mov demuxer
works.  It felt so... Automatic. As if the structure in itself was so very self
sufficient, as if it had a mind of its own...

Okay, kidding aside, I finally saw what a demuxer of this scale actually looks
like. There are so many corner cases you need to take care of. It must be so
robust that it could parse the file even if it is corrupted in some way (up to
some extent). Muxers are easy, you write what exactly you want without worrying
much. Anyway, as of now I have to implement a very simple demuxer, and improve
it later. Till now I have implemented a few boxes, I guess I could finish the
rest by tomorrow.

Okay okay I know that previously a conversion test that I did from jpeg to heic
is a lossy to lossy conversion (because jpeg itself contained a lossy compressed
image), and might not be the best way to test it, but to still be able to
compress that much further was indeed astonishing.

### June 15: Saturday

The demuxer is almost complete, the very basic version. But I might have to take
care of a lot of things before it can actually work. As of now there is a seg
fault somewhere, but I guess I know where it is. Too tired to look now though.

Okay, I wasted a lot of time thinking about the best way of implementation,
it was a bit complicated how to setup different boxes here, very unlike the
muxer. In many cases confused between mov_read_default or individual parse
function. Some parts of it I wrote nicely some are incomplete. There are still
so many things to test that I am not sure will work. Will continue tomorrow.

I am going on a vacation for 6 days the day after tomorrow. So might not be able
to write much or everyday on logs, but I'll still be working, just fewer hours
whenever I can.

### June 16: Sunday

Debugging is damn frustrating, but gdb makes it a lot easier with tonnes of
awesome features it provides. Easily finding seg faults, inspecting variables,
conditional breakpoints, and what not. Getting comfortable with gdb has always
come in handy for me.

Anyway, so I found the seg fault, it was what I thought it was. Fixed it, other
errors started to appear though. After some investigation I found all the
offsets are shifted. Somewhere there is a bad piece of code that does that, or
maybe absence of some code. Still couldn't find where it went wrong...

### June 19: Wednesday

It's driving me crazy, still couldn't find where the problem is. Everything
seems to be working fine. And what's amusing is the value of a variable that is
read from the file is random every time I run ffmpeg. I have no clue where
this random element could come from, some really weird thing is happening
somewhere. Although I have narrowed down the portion to search in. Let's see
what I can find.

### June 21: Friday

All this time I was searching everywhere, went through every line of code in
absolute detail, debugging away... turns out I just forgot to initialize ONE
LITTLE VARIABLE to zero! This is embarrassing. I wasted so much time on this I
want to hit myself. I could also have found this earlier if only I could have
been more careful while debugging. I thought when inspecting a variable in an
assignment instruction, it is already updated, whereas the shown statement
actually executes after we proceed to the next instruction (hence I kept seeing
garbage values all throughout the program). And also of course I am on a
vacation so I didn't get much time, when I did I was really tired of travelling
and all, even now I am. But anyway, it finally worked, demuxed the file I muxed
before nicely.

To finally witness the image or video played by ffmpeg is quite rewarding,
and really worth it.

### June 30: Sunday

In the last week I was mostly investigating why the heic file produced by the
muxer didn't work on MacOS. The heic files produced in mac were almost identical
to the one our muxer produced. It took a lot of time, I was stuck again, but
then I realised (Carl also helped) that colr tag and pixi tags were required for
some reason, even though nothing such is mentioned in the standard (afaik).

I would have implemented them before whatever the case, but the colr tag was a
bit tricky, and so I kept looking for a better option. The colr tag function
already present in the movenc.c did not support generating and writing icc
profiles (which were required for "prof" colour type, which in turn was required
for Apple compatible heic files). It could have been a lot of work to implement
that, I didn't even know how to. But then I tried a dirty hack of just copying
the colr tag from existing files and hard coding them into the muxer, and it
worked perfectly! The muxer finally produced Apple compatible heic files! But
of course I have to find a better way...

### July 1: Monday

Carl sent me a sample without colr atom, which was compatible with Apple. Now I
did have other samples where there was no colr atom which worked as well, but
looking at that file I realised what the reason could be, and I was right. Looks
like placing the iloc atom before certain atoms is the solution, it was that
simple (but not so easy to guess as nothing such is mentioned in the standard).

So I don't need to write the hard coded colr tag anymore, I can still produce
Apple compatible files. Muxer is pretty much done now. I will have to focus on
the demuxer henceforth (which is much important). Specifically I have to think
on how to manage tiles, which is, I guess, the most challenging part of the
project.

### July 2: Tuesday

I did some research on tiled heif files. Looked through old discussions in
projects like imagemagik and gpac. They have already added working support for
heif, and reading through those discussions gave me a lot of insight on how
Apple did things with their files.

Looks like Apple heic files typically divide an image into 48 tiles (8x6), each
usually of size 512x512, sequentially ordered from left to right and top to
bottom. If the full image does not have an even size, some part in the left edge
and bottom edge is apparently left blank, and needs to be cropped (not sure
about this but found it in a discussion). These tiles are also individual hevc
encoded frames and hence will need to be decoded individually and then stitched
up to get the full image. Sounds pretty simple but actually implementing it is
another story. 

Anyway, will have to do some more investigation, and maybe discuss things with
other developers before I start implementing. 

### July 4: Thursday

I had some questions so I asked the community. My first doubt was weather I
should create different streams for each tile or somehow keep all the tiles (as
frames) in a single stream. They (including Carl) thought that the former was a
bad idea. I didn't know how to implement the latter, I will have to research
some more. They also suggested that for now I need to write a new side data type
(in libavutil/frame.h) and make the demuxer output frames from tiles. They
mentioned some other things (which I didn't quite understand yet), but all in
all I understood that it was possible to pass the information such as final
image resolution, the tile resolution, the tile pixel format, and the tile
location along with each frame (as side data) so that the hevc decoder can
reconstruct the final image. Now how to actually do that is something I don't
know yet, will have to find out. It also seemed like some modification to the
decoder itself might be needed, I am not really sure though.  

### July 5: Friday

I improved the demuxer a bit. Added code to handle the cases where either iloc
would appear first or iinf, along with other changes. Right now I am also trying
to decide how to handle the item association logic. There is one way, which was
done in a previous patch, which I also thought to use, but that was kind of
specific to having different streams for different items. And that is not an
implementation that will work in the long run. I am trying hard to think of
another way but I guess before deciding that I would need to discuss things with
Carl, get his ideas on it.

### July 7: Sunday

I cannot contact Carl, maybe he is busy right now. Until I talk to him I don't
think I'll be able to do much at this point. I did some more changes on the
demuxer. Right now the demuxer is able to read the files created by me (simple
heif files with single images) and I am also trying to extend it to read the
images with with tiles (original images in the ticket). I read the the patch
posted a few years ago, and I found the item association logic in it was good
enough and (most importantly) working. So I copied the whole iprp function from
it (please don't judge me :/) and made changes in my code to work with it. There
are a few problems though, I'll take care of them tomorrow.

Apart from that I have also been looking around the code base for what the other
developers suggested, and its getting more and more confusing, I am really not
sure what to do.

### July 8: Monday

A small success! I finally made it work with the files with tiles (with some
hacks here and there). The demuxer can now read tiles, one in each stream as
attached pics. Although the hacks I did are quite problematic and should be
taken care of soon. 

The iinf tag consists of a list of all items, with their types (hvc1, grid and
Exif). As of now I am creating the streams within this tag. But the problem was
that only 1 through 48 items were the tiles (hvc1 type). Others were a "grid"
item which consisted grid info, a thumbnail and Exif item for exif metadata. I
had to hard code things to get only the tiles and ignore others. I did these
"hacks" at other portions (iloc, iprp) as well. All in all I need to design a
better structure that doesn't need these hacks. Also, the grid item has mdat
offset values instead of file offset values, and as we don't know the location
of mdat until the very end I can't figure out what to do about that.

### July 9: Tuesday 

I was finally able to talk to Carl today. He suggested that we could somehow
inform the calling application (ffmpeg here, which works on libav\*) that the
frames (tiles) are the part of a bigger picture and the application should do
something about that. This is one possibility, the other being the "side data to
decoder" approach, which could be more difficult. He also mentioned that I
should focus on keeping the tiles not in different streams but in a single
stream as if they were part of a movie. I have started working on it. I guess I
could use add_index_entry to add each tile as a frame to a single stream.

### July 10: Wednesday 

I used the add_index_entry approach. I guess I did successfully read all the
tiles into a single stream. Although to do this I had to strip off the current
structure which was based on different streams. There as still some things which
are hard coded but it is better than before. I am still using attached pic
approach for the simpler files (without tiles) so they still work with it. Using
index entries only with files with tiles. I am not sure if I am using the index
entries correctly though, even though I checked entries are correctly added, I
am not able to see any frames with ffplay, it doesn't display anything. There
are no errors either. Maybe it is discarding the frames or something I don't
know but there is something wrong, I'll have to consult Carl again maybe :/

### July 11: Thursday

Hmm... Strange. I sent the logs from before to Carl and he said that only one
frame is decoded. But I am pretty sure I set up the index correctly, why then
are not all the tiles decoded? I sent a mail to Carl, meanwhile I'll see if I
can figure it out myself...

### July 12: Friday 

After hours of debugging I finally figured it out (at times like this I feel
like Sherlock Holmes trying to find the killer, only he seems to know what he is
doing :/ ). So I just had to initialize the MOVStreamContext::pb with the
AVFormatContext::pb. Now I could see all 48 tiles one by one as if in a
slideshow on ffplay. Progress!

This was a milestone I guess. Next step would be to somehow pass these tiles to
the calling application and inform it to stitch them together. I haven't worked
at this level of application before so don't really know the details of it.
This was Carl's idea, I hope he sheds some light on it.

### July 13: Saturday

I used ffmpeg to split the tiled heic files into multiple 'tiles%d.jpg'. I found
ffmpeg already has a tiles filter which can be used to stitch tiles together to
reconstruct the full image. I found the command to do that and voila! I had the
full image as output!

So at this stage the demuxer is still useful, one can use the above commands to
get the desired image. But of course a better solution would be to somehow make
ffmpeg do all this automatically for files with tiles. The image would also need
to be cropped to its actual resolution and rotations (if any) applied.
Considering all these it looks like letting the calling application handle tiles
was a good suggestion that Carl made.

Currently, I am browsing through the ffmpeg application code, and waiting for
Carl's reply of course.

### July 14: Sunday

In the process of hard coding certain things for tile management, I managed to
break the demuxer for simpler files :( So I got rid of those, considered some
corner cases and as of now it works like a charm with any file I throw at it :)
Of course I only have a few samples but still this is the first time all of them
worked on my demuxer. I will have to get more samples soon though.
