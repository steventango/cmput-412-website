---
layout: post
title:  Final Project - Autonomous Driving with Duckietown
redirect_from:
- /final-project/
---

<div style="align-items: center; display: flex;">
      <img src="/cmput-412-website/images/final-project/nine_ducks_on_two_dividers.avif" alt="Nine ducks on two dividers" style="width: 49%;">
      <div style="display: inline-block; width: 48%;">
          <img src="/cmput-412-website/images/final-project/three_ducks_on_a_windowsill.avif" alt="Three ducks on a windowsill">
          <img src="/cmput-412-website/images/final-project/six_ducks_on_two_windowsills.avif" alt="Six ducks on two windowsills">
          <img src="/cmput-412-website/images/final-project/bam.avif" alt="Bam!">
      </div>
</div>

## Round 1

This was our best round, where we actually managed to park help-free, but had
some issues with lane following as our understanding of the "stay within the
correct lane" requirement assumed we could touch the yellow line, but turns out
we can't.

<iframe
      width="100%"
      height="315"
      src="https://www.youtube.com/embed/HpoDrbf7JZs"
      title="YouTube video player"
      frameborder="0"
      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
      allowfullscreen>
</iframe>

## Round 2

We did better on not touching the yellow line, but as we hastily adjusted the
tuning, the bot would lose the line and then go off the road. We also had some
issues with the Duckiebot's motors not turning as fast (likely as the bot's
battery percentage had gone down a bit), so we had issues with parking the bot
effectively.

<iframe
      width="100%"
      height="315"
      src="https://www.youtube.com/embed/tiARxHdgdK8"
      title="YouTube video player"
      frameborder="0"
      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
      allowfullscreen>
</iframe>

## Round 3

We achieved similar results to round 2. In interesting issue is that we did not
park as well in parking stall 2 and 3, in machine learning terms, we should have
overfitted harder to the testing distribution. We only had the time to tune the
parking parameters for stall 1 and 4, so the parking behaviour on stalls 2 and 3
was untested prior to the demo.

<iframe
      width="100%"
      height="315"
      src="https://www.youtube.com/embed/XmtXdPu80Jo"
      title="YouTube video player"
      frameborder="0"
      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
      allowfullscreen>
</iframe>

## Stage 1 - Loops and lane following

### Implementation

Akemi mostly worked on this part. After trying to be too clever, he switched
over to a simple strategy the day before the project was due, having thought up
the approach in the shower.

I started with the lab 4 code, which Steven had set up to read apriltags a month
back. I removed all the stale timeouts and distance measures of the apriltag in
the robot frame, storing just whatever was the latest seen apriltag on the
node's instance. With a red-line detection, the bot switches an attribute to True,
when it sees a sufficient "amount" of red line in its camera. Then, if the
attribute is True and we no longer see the red line, that must mean the
duckiebot is almost perfectly positioned with its wheels on the line. At this
point, we look up the latest apriltag and make a hardcoded forward/left/right
movement to get through the intersection. I was sort of surprised how well this
worked right away.

### Challenges

Most our problems came from mass murdering the duckiebots... We ended up with 4
in our fleet, since something we did to them kept breaking them in different
ways. Don't worry, we asked for permission or made sure the bots weren't being
used in all cases

 - `csc22902`: Failed to boot. It'd get to the white light stage and just say
   there for hours
 - `csc22920`: (We used this one for the demo). This bot was unable to build
   using `dts`, requiring us to extract the docker command used by duckietown
   shell and run it directly
 - `csc22927`: Right wheel started clicking and getting stuck. We swapped the
   wheel with `csc22902`'s, but that seems to have spread the ram-infection
   issue from below
 - `csc22933`: Ram overflow caused docker to be unable to build at all. We could
   fix this through `systemctl restart docker.service`... though that would
   break the camera container until we `systemctl reboot`. The whole process
   takes about 7-10mins, so we stopped using this bot

Now the problem with mass-murder as seen here is that the tuning between bots
was completely different. To make matters worse, tuning on a duckietbot with
<60% battery consistently gave completely wrong numbers when the bot got fully
charged again. This tuning issue resulted in our hard-coded turns sometimes
going into the wrong lane, before lane following was re-enabled to correct them.

## Stage 2

### Implementation

First, Akemi has had it with the make-shift hsv masking we were constantly
doing, so he quickly built it out to a proper software tool. *[Available
here](https://github.com/Aizuko/hsv_tool) for an unlimited time only! Get it
while it's still uncommented!*

Using this tool and the picture-saving node from lab 5, we quickly found HSV
masks that were strong in all conditions, even working in both rooms, for the
crossing duckies. We then used a jupyter notebook to guesstimate the plan:

 1. Get a callback from the camera and crop it to about at 150 pixel strip,
    roughly in the middle on the image
 2. Apply our hsv mask
 3. Flatten the image into a single channel greyscale, using the hsv mask
 4. Turn all pixels picked up by the hsv mask to 1 and all others to 0
 5. Sum the pixels and compare with the threshold number we set. We found about
    8000 was pretty good in the end

We only do this callback if the last seen apriltag is a crossing one. If this
callback returns true, the wheels immediately get killed. Then we scan every
second to check if there are any duckie crossing, or rather if the sum of the
hsv mask pixels is higher than the threshold. If we get 3 consecutive 1s
intervals saying it's safe to drive, then we go through the intersection and
switch to lane following. We used 3 intervals to fend off any possible noise
from the camera, especially important when duckie lives are on the line.

For the going-around the broken duckiebot, we again simplified the lab 4
following code to a single boolean indicating whether another bot is visible.
When it flips to true, the state machine initiates a sequence of completely
hard-coded movements to get around the bot, which actually ended up working
pretty well.

### Challenges

The biggest challenge was Akemi not feeling like reading the rubric. Notably he
missed the important bit about having to stop at all crossings, and not just
charging through if there aren't any duckies in the way... Oops

We also initially tried to use English driving to get around the broken
duckiebot, but we kept running into issues with the bot doing a 180 in its
current lane and just driving us back to stage 1. The more we hard-coded this
section, the more reliable our going-around got, so in the end we ended up with
a complete hard-code that performed consistently.

## Stage 3

### Implementation

Steven initially decided to take the computer vision "dark magic" he learned in
Martin's infamous CMPUT 428, to fuse measurements the time of flight (TOF)
sensor, apriltag detector, camera, and odometry to get perfect parking every
time. We used the camera to both calculate our pose relative to the april tags
and by computing the vanishing-point of the yellow parking lines to attempt to
center ourselves in the lane. Unfortunately, not only is this quite complex
task, the camera sensors and apriltag detections were not reliable enough. Our
final solution ended up with __just__ the TOF.

Now all the TOF gives is a fairly accurate direct-forward distance, which is
problematic when we need to figure out our pose in the parking lot. With some
clever thinking, Steven made the bot systematically wiggle left and right, until
it detected the apriltag opposite of the entrance with the TOF sensor! This
would give a good distance estimate relative to that apriltag. After getting
sufficiently close to it, the bot would turn towards the parking stall opposite
to our desired one. Next it again did a wiggle to find the apriltag in the
opposite stall, aligning itself such that the TOF sensor reads the minimum
distance. This allowed us to tell the apriltag apart from the wooden-backboard.
Once aligned, we just just reverse into the parking stall, until the TOF sensor
read a distance over 1.15 m (the TOF sensor goes out of range after
approximately 1.20 m). We then blindly reversed a bit more to make sure we were
fully in the stall.

### Challenges

A major challenge was that there was not a lot of time to tune the parking
parameters. We had only the time to tune for parking stalls 1 and 4, and we were
tested on parking stall 1 which we tuned for and parking stalls 2 and 3 which
were completely untested. If we had more time, we would have tuned for all
stalls so that we could have a more reliable parking solution.

Particularly on `csc22920`, the left-turning was MUCH stronger than
the right turning, which meant we needed to use a very asymmetric force between
the two directions. We found right turns that used an omega 3 times higher than
the left seemed to truly balance things out.

Unlike stage 1 though, Steven pulled off a miracle and blindly guessed the
correct tuning parameters without testing seconds before our first demo. It
struggled to work well 2 minutes before our demo, and the blind-guessed numbers
he chose at put in worked perfectly!

## Sources

- [Find Middle of Line Using Moments](https://stackoverflow.com/questions/64396183/opencv-find-a-middle-line-of-a-contour-python)
