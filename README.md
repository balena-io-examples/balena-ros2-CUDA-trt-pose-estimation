# balena ROS Jetson TRT Pose Estimation container

[![balena deploy button](https://www.balena.io/deploy.png)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/balena-io-examples/balena-ros2-CUDA-trt-pose-estimation/)

## About
This repo installs [ROS Eloquent](https://docs.ros.org/en/eloquent/Releases.html) into a 64-bit Ubuntu Arm container and runs live inferencing doing human pose detection and estimation using a camera, ready to be deployed on an NVIDIA Jetson device running balenaOS.  This repo specifically makes use of CUDA, so, no other device types (Raspberry Pi, Beagle, etc) will work.  The Dockerfile adds the basic requirements, adds the ROS key and APT sources, installs the ROS binaries and packages, then brings in OpenCV, CUDA, PyTorch, and the rest of the neeeded components.  Also keep in mind that this repo sets up the ROS Eloquent "Desktop" install, which is a fully featured (i.e., large) installation including a desktop GUI and Rviz for visualization.  If you only need the minimal ROS installation, there is also a "Base" repo available here: https://github.com/balena-io-examples/balena-ros2-foxy-base, which makes for a slimmer container, but that does not include any CUDA tooling or computer vision capability out-of-the-box.

## Usage
To get started, you can simply click the "Deploy with balena" button above, or use the traditional workflow and do a `git clone` of this repo, and then a `balena push YourAppNameHere` to deploy the container to your device.  You can read about how to create a balena application, setup your device, and install the balena CLI here:  https://www.balena.io/docs/learn/getting-started/jetson-nano/python/

Once your device is provisioned and online, and the containers have been built and downloaded, you are ready.  This repo will build a full desktop environment, so you'll want to attach a monitor, keyboard, mouse, and a USB camera to your Jetson.  Also take note, this container is about 10gb in size, so it will take a long time to build, and then you have a 10gb download, so this could be a multiple-hour exercise.  Have some tea ready.

Once complete and the container is downloaded, the Jetson will automatically start the Desktop and the Pose Estimation application will launch.  Rviz will start up fairly quickly, but, the AI model will automatically download and then install, and that will take about 5 to 7 minutes.  This only occurs the first time, and subsequent launches are faster.  Once complete, aim the USB webcam at yourself, and you should be detected and represented in real-time!

## Conclusion
This repo is designed to get you up and running with ROS + CUDA quickly, and help you to get all of the artifacts installed and functional.  This pose estimation is of course just a demo of the functionality, but hopefully it helps you to iterate and deploy your own solutions quickly and easily.  You can find [more documentation on ROS here](https://docs.ros.org/en/eloquent/Tutorials.html).



