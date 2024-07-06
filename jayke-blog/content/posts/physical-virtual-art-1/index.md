+++
title = 'Making virtual art physical - Part 1'
date = 2024-07-06T13:54:52+02:00
draft = false
keywords = ['electronics', 'code', 'open-source']

[params]
  math = true
+++
Time for my first actual post! Let me tell you about my latest hobby project. This is a part 1 in a two-part series -
strap in!

## Drarwing

About two months ago, my colleague [Berry](https://github.com/berryvansomeren) shared his latest project:
[Drarwing](https://www.berryvansomeren.com/posts/drarwing). I wholeheartedly suggest reading the blog post (and the
subsequent post about how he turned it into a web-hosted tool
[here](https://www.berryvansomeren.com/posts/drarwing_web)), but in short, Drarwing takes a photo, and using a
generative approach, turns it into a painting. It semi-randomly places brushstrokes on the canvas, and checks whether the
resulting image is more like the original than before. If it is, the brushtroke is kept, otherwise it tries again with
a different random stroke. That results in beautiful painting-like images, but what is _really_ cool is watching the
process unfold:
![A gif displaying the drarwing process in action](assets/drarwing.gif "Mesmerizing: Watch drarwing create a painting")
As we were talking about it, we started thinking. What if you would continue drawing the next picture, once the previous
one is done? That way you would see the picture morph from one into the next. And if you have that, wouldn't it make for
a really cool screensaver? And if you have _that_, what about putting it in a picture frame and making it look like a
real painting? A plan was born.

## The plan

Unfortunately, Berry was a little preoccupied with becoming a father (_priorities_, right?), but he graciously said
_"The code is open source so feel free to run with those ideas yourself!"_. And with that, my new hobby project was
born. There are only a few small things that need to be done:

- The application needs to continue with the next photo once the previous is done
- It needs to be run on some hardware small enough to fit in a frame, so the code needs to be _fast_
- There should be a convenient way to get the photos on the device, without physical interaction
- We need to create a frame that has a display, contains all the components **and** looks like a painting

Piece of cake.

## Modifying the code

Lets start with the part that has the lowest barrier of entry (I'm slowly coming to graps with how my brain works):
Modify the code. I forked Berry's repository, and started experimenting. Naturally, the version I created is available
as [open source](https://github.com/JaykeMeijer/drarwing_continuous), like the original. Read on to see the changes I
made.

Modifying the code to draw the next picture once the previous is done was the first step, and it was very straight
forward - the joys of working with code made by an experienced software engineer. To follow some best practices from the
world of rendering and graphical user interfaces, I created a separate rendering thread to have control over the
framerate, decoupled from the speed of the drawing. I also introduced some parameters to control how fast the drawing is
done and how long the result is shown before starting the next drawing. Finally, I introduced some debugging
functionality that allow showing the "difference image" (the difference between the current painting and the target
image) and the target image itself.

### FASTER!

The speed of the drawing was the next step. Since we'll be using low-powered hardware and we need to add some complexity
(more below), I wanted to find all the performance I could. That calls for something I'm a big advocate of:

:star2:PROFILING:star2:

(this might be a future blog topic). It immediately became clear that almost half the time was spent copying objects.
Some further investigation allowed me to cut this time down significantly, by decreasing the number of `deepcopy()`s
required, and using `numpy`s built-in copy mechanism for the `ndarray`s containing the image data instead.
[These changes](https://github.com/berryvansomeren/drarwing_web/pull/1/files) were also merged back to the original
Drarwing application, allowing some more advanced features there too.

A further speed improvement is to work with a downscaled difference image (1/4th of the actual image size). The brush
strokes are sufficiently large that using a higher resolution does not improve the result.

### How different are colours?

All this performance improvement means we now have the margin to use some more advanced scoring metrics. A scoring
metric is used to determine how good a randomly placed brushstroke really is. This is determined by looping through all
the pixels, and comparing how that pixel looks in the drawing versus the original image. This is then added up per pixel
to determine how much the entire drawing looks like the original.

In the original Drarwing, the difference between two pixels is simply taken as the absolute difference between their
grayscale values. This works fine, because the canvas starts as white, meaning that the entire picture will get covered
with some color closer to the target anyway.

However, in this version the base canvas is not white, but it's the previous drawing. That causes some issues, as two
colors that look completely similar in grayscale, may look completely different to us. This can lead to large areas of
the image having the wrong color.

To fix this, we can use something called the **ΔE metric**. This metric works in the
[CIELAB color space](https://en.wikipedia.org/wiki/CIELAB_color_space), which is specifically designed to represent how
colors are perceived by humans. We can then use the CIE1976 calculation to determine the difference between two colors
as follows:

$$
\Delta E = \sqrt{(L_{2}-L_{1})^{2}+(a_{2}-a_{1})^{2}+(b_{2}-b_{1})^{2}}
$$

(Of course, since we're using Python, there's a
[package for that](https://colour.readthedocs.io/en/latest/generated/colour.difference.delta_E_CIE1976.html)). Note that
there are more advanced implementations of the ΔE metric, but these have only small improvements that don't have a lot
of impact, while they do add a lot of performance costs.

### I want to see some footage

In the end, the result looks like this. Note the rendering speed (on a high-power desktop), the reasonably good coverage
of the previous painting - most of the time - and in the second half of the video the switching between original, target
and difference images:

{{< youtube weyghflwnj8 >}}

## Scale it down

Of course, we can't keep running this application on a high-performance desktop machine. Ideally, the entire device will
be self-contained, only needing power from the outside. So, we need a _small_ computer. In today's world, that means a
Raspberry Pi. And as smaller is better in this case, I decided to go with a Raspberry Pi Zero 2 W. A bonus here is that
this version has Wi-Fi included, which will be very convenient later on.

### Keeping control

The Raspberry Pi does have a limitation though: It does not have a power button. This is fine for devices used as an
always-on server, or when running the device interactively. For this use-case however, it means it can't be shutdown
properly. That leaves just pulling the power, which can lead to SD card corruption and a bricked Raspberry installation.
Not ideal.

So we need a solution. The Pi supports a "halt state", in which it is in a sort of deep-sleep state (and won't corrupt
the storage when powered off). While in this state, if you short GPIO3 to ground, it will wake back up.

So [the solution](https://howchoo.com/pi/how-to-add-a-power-button-to-your-raspberry-pi/) is simple: We add a button
between GPIO3 and GND. This button can be used to power the Pi back up, and by adding a simple script that listens on
this pin while the Pi is turned on, it can be used to put the Pi into the halt state.

A second button is added to trigger Drarwing to start rendering the next image right away, which seemed like a
fun additional functionality.

### Pushing pictures

The "self-contained" part of the frame also means I want to have a clean and simple way to update the photos that are
shown on the device. This is where the Wi-Fi functionality comes in. As the Pi is connected to the internet, it was
[fairly trivial](https://www.thedigitalpictureframe.com/how-to-synchronize-your-raspberry-pi-digital-picture-frame-with-google-drive-using-rclone/)
to add a sync with a Google Drive folder. It uses `rclone` to create a clone of a folder in my Google Drive, meaning
that it's really easy to add or remove photos from the frame.

## Time for a break

As I started writing, I realized I have a _lot_ to say about this project. We haven't even started on the exciting part!
As the software side is done now, I'm going to leave the hardware part for a next post. To be continued.
