# Exercise 3 - Computer Vision for Robotics

## Challenges

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

## Sources

* <https://docs.duckietown.org/daffy/duckietown-classical-robotics/out/cra_basic_augmented_reality_exercise.html>
* <http://wiki.ros.org/cv_bridge/Tutorials/ConvertingBetweenROSImagesAndOpenCVImagesPython>
* <https://github.com/duckietown/dt-duckiebot-interface/blob/daffy/packages/camera_driver/>
* <https://docs.ros.org/en/api/image_geometry/html/python/>
