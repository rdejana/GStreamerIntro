# GStreamer Introduction and Demo
The following is a simple introduction and demo to GStreamer, specfically on the Jetson Xavier NX platform.
Additional documentation can be at https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/accelerated_gstreamer.html

The majority of this examples will be use the GStreamer tool gst-launch-1.0.

## Part 1: Our first pipeline.
Our first pipeline will be a very simple test image.

If you are running from your NX, run the command: `gst-launch-1.0 videotestsrc ! videoconvert ! xvimagesink`.  For other platforms, e.g. macOS, `gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink` should work.

In this example, `videotestsrc` is a video test source.  Using `gst-inspect-1.0` you can see that there is the ability to adjust the pattern.  For example, changing to `gst-launch-1.0 videotestsrc pattern="circular" ! videoconvert ! xvimagesink`


A more complicated version of this is: `gst-launch-1.0 videotestsrc ! nvvidconv ! 'video/x-raw'  ! nvvidconv ! nvegltransform ! nveglglessink -e`.  
Unlike the first example, this one leverages a number of Nvidia plugins:
- nvvidconv: video format conversion and scaling
- nvegltransform: Video transform element for NVMM to EGLimage.  nveglglessink only. 
- nveglglessink: EGL/GLES videosink

## Part 2: USB Camera
To use a USB camera on the Jetson, you need to use the plugin v4l2src.  While not covered here, if you are using a Raspberry Pi Camera, you would need to use the plugin nvarguscamerasrc.

## Part 3: Not just video, let's look at audio


## Part 4: Streaming
abc
## Part X: Using Gstreamer with OpenCV and Python.




sudo apt-get install v4l-utils
