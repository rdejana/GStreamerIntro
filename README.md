# GStreamer Introduction and Demo
The following is a simple introduction and demo to GStreamer, specfically on the Jetson Xavier NX platform.
Additional documentation can be at https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/accelerated_gstreamer.html

The majority of this examples will be use the GStreamer tool gst-launch-1.0.

## Part 1: Our first pipeline.
Our first pipeline will be a very simple test image.

If you are running from your NX, run the command: `gst-launch-1.0 videotestsrc ! videoconvert ! xvimagesink`.  For other platforms, e.g. macOS, `gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink` should work.

In this example, `videotestsrc` is a video test source.  Using `gst-inspect-1.0` you can see that there is the ability to adjust the pattern.  For example, changing to `gst-launch-1.0 videotestsrc pattern="circular" ! videoconvert ! xvimagesink`


A more complicated version of this is: `gst-launch-1.0 videotestsrc ! nvvidconv ! nvegltransform ! nveglglessink -e`.  
Unlike the first example, this one leverages a number of Nvidia plugins:
- nvvidconv: video format conversion and scaling
- nvegltransform: Video transform element for NVMM to EGLimage.  nveglglessink only. 
- nveglglessink: EGL/GLES videosink

## Part 2: USB Camera
To use a USB camera on the Jetson, you need to use the plugin v4l2src.  While not covered here, if you are using a Raspberry Pi Camera, you would need to use the plugin nvarguscamerasrc.

gst-launch-1.0  v4l2src device=/dev/video0 ! xvimagesink

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,format=YUY2 ! videoconvert ! video/x-raw,format=GRAY8 ! videoconvert ! ximagesink
gst-launch-1.0  v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,format=YUY2 ! videoconvert ! video/x-raw,format=GRAY8 ! videoconvert ! video/x-raw,format=BGRx ! ximagesink

v4l2src device=/dev/video0 ! nvvidconv ! nv3dsink
v4l2src device=/dev/video0 ! nvvidconv ! nvegltransform ! nveglglessink -e
gst-launch-1.0  v4l2src device=/dev/video0 ! nvvidconv ! 'video/x-raw(memory:NVMM),height=200,width=300,format=I420' ! nv3dsink

## Part 3: Not just video, let's look at audio
## Part N: Fun and games

Special effects

gst-launch-1.0  avfvideosrc device-index=0 ! videoconvert ! warptv ! videoconvert ! autovideosink

gst-launch-1.0 -v videotestsrc ! agingtv scratch-lines=15 ! videoconvert ! autovideosink

gst-launch-1.0 videotestsrc ! video/x-raw, format=GRAY8 ! videoconvert ! autovideosink 

vs 

gst-launch-1.0 videotestsrc ! video/x-raw ! videoconvert ! video/x-raw,format=GRAY ! autovideosink

Which?  Depends what you need.  Often lower down is better, but what if you want...

gst-launch-1.0 videotestsrc ! queue ! tee name=t  t. ! queue ! autovideosink sync=false t. ! queue ! videoconvert ! video/x-raw,format=GRAY8 ! autovideosink sync=false -e


now with video

gst-launch-1.0  avfvideosrc device-index=0 ! videoconvert ! agingtv scratch-lines=15 ! videoconvert ! autovideosink

gst-launch-1.0 -v videotestsrc ! agingtv scratch-lines=15 ! videoconvert ! autovideosink

gst-launch-1.0 videotestsrc ! videobalance  contrast=1.5 brightness=-.3 saturation=1.2 ! videoconvert ! autovideosink

gst-launch-1.0 avfvideosrc device-index=0 ! videobalance  contrast=1.5 brightness=-.3 saturation=1.2 ! videoconvert ! autovideosink

gst-launch-1.0 avfvideosrc device-index=0 ! videobalance  contrast=1.5 brightness=.1 saturation=1.2 ! videoconvert ! autovideosink


Some text: gst-launch-1.0  avfvideosrc device-index=0 ! videoconvert ! textoverlay text="Hello World" valignment=bottom halignment=left font-desc="Sans, 40" ! autovideosink

gst-launch-1.0 videotestsrc ! queue ! tee name=t  t. ! queue ! autovideosink sync=false t. ! queue ! videoflip method=horizontal-flip ! autovideosink sync=false -e

gst-launch-1.0 avfvideosrc device-index=0 ! queue ! tee name=t  t. ! queue ! autovideosink sync=false t. ! queue ! videoflip method=horizontal-flip ! autovideosink sync=false -e


Add an image one

Image merge example (see below)


## Part 4: Streaming
abc
## Part X: Using Gstreamer with OpenCV and Python.




gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! matroskademux name=d ! queue ! vp8dec ! videoconvert ! autovideosink d. ! queue ! vorbisdec ! audioconvert ! audioresample ! autoaudiosink

gst-device-monitor-1.0 Video/Source
  video/x-raw, format=(string)YUY2, width=(int)640, height=(int)480, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 25/1, 20/1, 15/1, 10/1, 5/1 };

gst-launch-1.0 -e videomixer name=mix \
    ! xvimagesink \
    videotestsrc\
            ! video/x-raw, framerate=10/1, width=640, height=360 \
            ! mix.sink_0 \
    videotestsrc pattern="snow" \
            ! video/x-raw, framerate=10/1, width=200, height=150 \
            ! mix.sink_1
            
            
  gst-launch-1.0 -e videomixer name=mix \
    ! xvimagesink \
    videotestsrc\
            ! video/x-raw, framerate=10/1, width=640, height=360 \
            ! mix.sink_0 \
    v4l2src device=/dev/video0 ! video/x-raw,framerate=10/1,width=200, height=150  \
            ! mix.sink_1


gst-launch-1.0 videotestsrc !   'video/x-raw, format=(string)I420, width=(int)640, \
  height=(int)480' ! omxh264enc !   'video/x-h264, stream-format=(string)byte-stream' ! h264parse ! omxh264dec ! nveglglessink -e
  
  gst-launch-1.0 v4l2src device=/dev/video0 ! omxh264enc !   'video/x-h264, stream-format=(string)byte-stream' ! h264parse ! omxh264dec ! nveglglessink -e
  
  
gst-launch-1.0 nvcompositor name=mix sink_0::xpos=0 sink_0::ypos=0 sink_1::xpos=0 sink_1::ypos=240 ! nvegltransform ! nveglglessink videotestsrc ! nvvidconv ! mix.sink_0 v4l2src device=/dev/video0 ! nvvidconv ! mix.sink_1

gst-launch-1.0 nvcompositor name=mix sink_0::xpos=0 sink_0::ypos=0 sink_0::zorder=10 sink_1::xpos=0 sink_1::ypos=0 ! nvegltransform ! nveglglessink videotestsrc ! nvvidconv ! mix.sink_0 v4l2src device=/dev/video0 ! nvvidconv ! mix.sink_1
  
  
  

