# Exercise 3 - Computer Vision for Robotics

![AR Ducks](images/exercise-3/ar-ducks.png)

A screenshot of our [Unit A-4 Advanced Augmented Reality Exercise](https://docs.duckietown.org/daffy/duckietown-classical-robotics/out/cra_apriltag_augmented_reality_exercise.html)
results.

## Part One - Computer Vision

### Deliverable 1: April Tag Detection and Labeling

The following video depicts our apriltag detector image topic viewed with `rqt_image_view` demonstrating our apriltag node detecting several apriltags and labeling each with its bounding box and ID number.

<iframe width="100%" height="315" src="https://www.youtube.com/embed/gAck5-vHF6U" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### **What does the april tag library return to you for determining its position?**

<!-- TODO -->

### **Which directions do the X, Y, Z values of your detection increase / decrease?**

<!-- TODO -->

### **What frame orientation does the april tag use?**

<!-- TODO: https://docs.google.com/document/d/1bQfkR_tmwctFozEZlZkmojBZHkegscJPJVuw-IEXwI4/edit#heading=h.vzisu8fiu4hj -->

### **Why are detections from far away prone to error?**

Far away apriltags have fewer pixels to detect, so it is more prone to
error and unstable detection as can be seen in the video with apriltag ID 94.

### **Why may you want to limit the rate of detections?**

April tag detection is computationally expensive, so limiting the rate of
detections can reduce Duckiebot CPU usage. Furthermore, if the image does not
change much, it is can unnecessary to recompute April tag detections.

### Challenges

### `.dockerignore`

The `template-ros` repository has a `.dockerignore` and it ignores our
additional files and directories, we have to whitelist them like:

```.dtignore
!maps
```

### cv_bridge

From looking at the [camera driver code](https://github.com/duckietown/dt-duckiebot-interface/blob/daffy/packages/camera_driver/), I learned that we can use []`cv_bridge`](http://wiki.ros.org/cv_bridge/Tutorials/ConvertingBetweenROSImagesAndOpenCVImagesPython)
to convert between OpenCV images and CompressedImage messages.

## image_geometry

The [image_geometry](https://docs.ros.org/en/api/image_geometry/html/python/)
package seems useful for undistorting raw images.

### Docker 23.0.1 breaks dts devel run

After upgrading to Docker 23.0.1, `dts devel run` would error with message:
`docker: Error response from daemon: No command specified.`.

I had to [downgrade](https://docs.docker.com/engine/install/ubuntu/) to Docker 20.10.23 to get it to work again.

```bash
VERSION_STRING=5:20.10.23~3-0~ubuntu-focal
steven@steven-Ubuntu20:~/Github/duckietown/lab3/farfetched$ sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

## Part Two - Lane Following

Here are the videos!

 - [American driver](https://youtu.be/K4qqWjGXsec). 0.3 velocity, P-controller,
   drives on the right
 - [English driver](https://youtu.be/lVeuNHGCy6w). 0.3 velocity, P-controller,
   drives on the left
 - [Drunk driver](https://youtu.be/Nt8Qxyd7FjQ). 0.6 velocity, P-controller,
   attempts to drive on the left

### Deliverable 2: Lane Following English Driver Style

### **What is the error for your PID controller?**

We use a multi-stage Opencv pipeline to process the image

 1. Crop out the top 1/3 and bottom 1/5
 2. Blur the entire image by 5 pixels in both directions
 3. Apply HSV thresholds to mask out everything except the yellow lane. We have
    two separate thresholds, one for each room. Both thresholds mask out ducks,
    though the csc229 ones doesn't correctly mask out the carpet in csc235
 4. Detect contours in the new black-white image
 5. Sort contours by area. We initially considered using the top-k contours and
    averaging them, though we found outlier contours can do a lot more damage
    this way, so we just used the single largest one instead
 6. Find the center of the largest contour
 7. Draw a line from the center of the largest contour to the right (left with
    English-driving) with a length of exactly the distance of the center point
    from the top of the image
 8. **Set the error** as the difference between the line's right-most point and
    the center of the image. This means the error is measured in pixels

### **If your proportional controller did not work well alone, what could have caused this?**

Initially, our controller failed to work since we published the images at 3Hz.
The logic is that opencv is a "heavy" process, so publishing at a high frequency
would just lead to images being discarded from the publisher queue. The system
suddenly started working, when we change this to 30Hz... at 0.3 velocity.

However, at 0.6 velocity, the P-term-only controller really struggled (see
[third video](https://youtu.be/Nt8Qxyd7FjQ)). Our P-term-only controller making
the English driver look more like the drunk driver. It didn't work well since a
P term fails to consider the momentum built up by the system. At 0.3 velocity,
there isn't enough momentum to effect our controller noticeably, though at 0.6
this momentum leads to noticeable oscillation and hard turning overshoots.

### **Does the D term help your controller logic? Why or why not?**

The D term was pointless at 0.3. Actually it was detrimental, since the logic
of the program got harder to debug. We did try to implement one when using 0.6
velocity, however with 2 minute build times, we weren't able to sufficiently
tune it. An untuned D-term was far worse than a tuned P-controller, mostly since
it kept fighting against itself on the turns, which made it end up off the
track.

### **(Optional) Why or why not was the I term useful for your robot?**

Given how problematic the D-term was, we never even considered adding an I-term
during this assignment. We doubt it'd be much help either, since there isn't
any steady-state error to correct against with an integral term when we're just
driving on a horizontal surface.

## Part Three - Localization using Sensor Fusion

### Deliverable 3: Record a video on your computer showing RViz: displaying your camera feed, odometry frame and static apriltag frames as it completes one circle. You can drive using manual control or lane following.

## **Where did your odometry seem to drift the most? Why would that be?**

<!-- TODO -->

## **Did adding the landmarks make it easier to understand where and when the odometry drifted?**

<!-- TODO -->

## Deliverable 4: Attach the generated transform tree graph, what is the root/parent frame?

The root/parent is `csc22927/footprint`.

Move the wheels and make note of which joint is moving, what type of joint is this?

`csc22927_left_wheel_axis_to_left_wheel` and `csc22927_left_wheel_axis_to_right_wheel`. The type of joint is
`continuous`. [urdf](https://github.com/duckietown/dt-duckiebot-interface/blob/daffy/packages/duckiebot_interface/urdf/duckiebot.urdf.xacro#L107)

You may notice that the wheel frames rotate when you rotate the wheels, but the frames never
move from the origin? Even if you launch your odometry node the duckiebot’s frames do not
move. Why is that?
1. Using a static transform attach your odometry child frame to the parent frame from the
URDF.
● What should the translation and rotation be from the odometry child to robot parent
frame? In what situation would you have to use something different?

Zero translation, zero rotation. When the parent frame and odometry frame are not identical we would have to use a more
complicated transformaton.

● After creating this link generate a new transform tree graph. What is the new root/parent
frame for your environment?
● Can a frame have two parents? What is your reasoning for this?
● Can an environment have more than one parent/root frame?

## Deliverable 5: Attach your newly generated transform tree graph, what is the new root/parent frame?

## Deliverable 6:
**Record a short video of your robot moving around the world frame with all the
robot frames / URDF attached to your moving odometry frame. Show the apriltag
detections topic in your camera feed and visualize the apriltag detections
frames in rviz.**

Questions
● How far off are your detections from the static ground truth.
● What are two factors that could cause this error?

Our detections are quite far off from the ground truth. Two factors that could cause this error are inaccurate odometry from wheel encoders and camera transform errors arising from camera
distortion.

Deliverable 7: Show a short video of your robot moving around the entire world (see
image below) using lane following and have your sensor fusion node teleport the robot if
an apriltag is found and use odometry if no apriltag is detected. Try to finish as close to
your starting point as possible.
Questions
● Is this a perfect system?
● What are the causes for some of the errors?
● What other approaches could you use to improve localization?

## Sources

* <https://docs.duckietown.org/daffy/duckietown-classical-robotics/out/cra_basic_augmented_reality_exercise.html>
* <http://wiki.ros.org/cv_bridge/Tutorials/ConvertingBetweenROSImagesAndOpenCVImagesPython>
* <https://github.com/duckietown/dt-duckiebot-interface/blob/daffy/packages/camera_driver/>
* <https://docs.ros.org/en/api/image_geometry/html/python/>
* <https://docs.docker.com/engine/install/ubuntu/>
* <https://github.com/duckietown/dt-core/blob/6d8e99a5849737f86cab72b04fd2b449528226be/packages/complete_image_pipeline/include/image_processing/ground_projection_geometry.py#L161>
* <https://bitesofcode.wordpress.com/2018/09/16/augmented-reality-with-python-and-opencv-part-2/>
* <https://einsteiniumstudios.com/beaglebone-opencv-line-following-robot.html>
* <http://wiki.ros.org/tf2/Tutorials/Writing%20a%20tf2%20static%20broadcaster%20%28Python%29>
* <https://nikolasent.github.io/opencv/2017/05/07/Bird's-Eye-View-Transformation.html>
* <http://wiki.ros.org/urdf/XML/joint>
* <https://github.com/duckietown/dt-duckiebot-interface/blob/daffy/packages/duckiebot_interface/urdf/duckiebot.urdf.xacro#L107>
* <https://wiki.ros.org/tf2/Tutorials/Writing%20a%20tf2%20listener%20(Python)>
