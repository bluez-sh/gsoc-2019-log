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
did have other samples where there was not colr atom which worked as well, but
looking at that file I realised what the reason could be, and I was right. Looks
like placing the iloc atom before certain atoms is the solution, it was that
simple (but not so easy to guess as nothing such is mentioned in the standard).

So I don't need to write the hard coded colr tag anymore, I can still produce
Apple compatible files. Muxer is pretty much done now. I will have to focus on
the demuxer henceforth (which is much important). Specifically I have to think
on how to manage tiles, which is, I guess, the most challenging part of the
project.
