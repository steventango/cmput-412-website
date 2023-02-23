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

## Docker 23.0.1 breaks dts devel run

After upgrading to Docker 23.0.1, `dts devel run` would error with message:
`docker: Error response from daemon: No command specified.`.

I had to [downgrade](https://docs.docker.com/engine/install/ubuntu/) to Docker 20.10.23 to get it to work again.

```bash
VERSION_STRING=5:20.10.23~3-0~ubuntu-focal
steven@steven-Ubuntu20:~/Github/duckietown/lab3/farfetched$ sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

## Sources

* <https://docs.duckietown.org/daffy/duckietown-classical-robotics/out/cra_basic_augmented_reality_exercise.html>
* <http://wiki.ros.org/cv_bridge/Tutorials/ConvertingBetweenROSImagesAndOpenCVImagesPython>
* <https://github.com/duckietown/dt-duckiebot-interface/blob/daffy/packages/camera_driver/>
* <https://docs.ros.org/en/api/image_geometry/html/python/>
* <https://docs.docker.com/engine/install/ubuntu/>
