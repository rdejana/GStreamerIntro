# Getting started Gstreamer on the Jetson NX

The following is a simple introduction and demo to GStreamer, specfically on the Jetson Xavier NX platform. Additional documentation can be at https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/accelerated_gstreamer.html

The majority of this examples will be use the GStreamer tool gst-launch-1.0.

## Part 1: 
Our first pipeline will be a simple video test image.
``
gst-launch-1.0 videotestsrc ! xvimagesink
``

This will display a classic "test pattern".  The command is composed of two elements, the `videotestsrc` and a video sink, `xvimagesink`.

Running `gst-inspect-1.0 videotestsrc` will provide some additional information on the src.  One of the properies we can set is the pattern.
``
gst-launch-1.0 videotestsrc pattern=snow ! xvimagesink and gst-launch-1.0 videotestsrc pattern=ball ! xvimagesink for example.
``

Nvidia also provides a couple of its own accellerated plugins:
- nv3dsink: a window-based rendering sink, and based on X11
- nveglglessink: EGL/GLES video sink
- nvvidconv: a Filter/Converter/Video/Scaler, converts video from one colorspace to another & Resizes
- nvegltransform: tranforms to the EGLImage format.

Inspecting nv3dsink, we can see that it requires an input of `video/x-raw(memory:NVMM)`.  
This is not someting that videotestsrc outputs, so we'll need to use nvvidconv to convert.
Inspecting this, we can see it can take `video/x-raw` and output  `video/x-raw(memory:NVMM)`.
```
gst-launch-1.0 videotestsrc ! nvvidconv ! nv3dsink -e
```

To use nveglglessink, we'll need to use nvvidconv and nvegltransform, to go from NVMM to EGLImage.
```
gst-launch-1.0 videotestsrc ! nvvidconv ! nvegltransform ! nveglglessink -e
```

Which sink to use?  Will it just depends.  xvimagesink is often easier to get going, but the nvidia ones provide additional acceration and perforance.
Note, there are additional Nvidia sinks that may be used, but may not work over technology like VNC, e.g nvdrmvideosink.

Part 2: USB Camera
Now that we have some experience, we'll add a camera to the fix.  We'll be using a USB camera, which leverages the v4l2src plugin.  While not covered here, if you are using a Raspberry Pi Camera, you would need to use the Nvidia plugin nvarguscamerasrc.

This example assumes your camera is using /dev/video0; you may need to adjust depending on your configuration.

As before, we can take advantage of a varity of sinks.

gst-launch-1.0 v4l2src device=/dev/video0 ! xvimagesink

gst-launch-1.0 v4l2src device=/dev/video0 ! nvvidconv ! nv3dsink -e

gst-launch-1.0 v4l2src device=/dev/video0 ! nvvidconv ! nvegltransform ! nveglglessink -e

Now that we have access to the cameara, we can explore what we can do.  Now the the camera is the limit; if your camera doesn't support 60 FPS and 4K, there is no use asking for it.  

To list all of your cameras, you'll run the command:

gst-device-monitor-1.0 Video/Source

And you'll get output similar to this (note, we are only concerned with video/x-raw in this case):

	name  : UVC Camera (046d:0825)
	class : Video/Source
	caps  : video/x-raw, format=(string)YUY2, width=(int)1280, height=(int)960, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 15/2, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 15/2, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)1184, height=(int)656, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)960, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)1024, height=(int)576, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)960, height=(int)544, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)800, height=(int)600, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)864, height=(int)480, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)800, height=(int)448, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)752, height=(int)416, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 25/1, 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)640, height=(int)480, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 25/1, 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)640, height=(int)360, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 25/1, 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)544, height=(int)288, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 25/1, 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)432, height=(int)240, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 25/1, 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)352, height=(int)288, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 25/1, 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)320, height=(int)240, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 25/1, 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)320, height=(int)176, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 25/1, 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)176, height=(int)144, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 25/1, 20/1, 15/1, 10/1, 5/1 };
	        video/x-raw, format=(string)YUY2, width=(int)160, height=(int)120, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 25/1, 20/1, 15/1, 10/1, 5/1 };
	        
You can see that for this camera, it's format is YUY2, and that our available dimensions and framerates are related.  Let's start by asking for 30 FPS.  Note, you'll need to use the X/1 for framerates.

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1 ! xvimagesink

Notice that the size of the window as changed.  Now what if we want 30 FPS at 1280 x 720?  Let's find out.

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=1280,height=960 ! xvimagesink

and for me, it fails.  If we look above, should be clear why.  Let's go the other way and ask for 160x120.

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=160,height=120 ! xvimagesink

And works as expected.   Feel free to explore what your camera can do!

Now what can we do with the video?  Say we want to see the image in grayscale.

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1 ! videoconvert ! video/x-raw,format=GRAY8 ! videoconvert  ! xvimagesink

Why is the extra videoconvert needed?  

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1 ! videoconvert ! video/x-raw,format=GRAY8  ! xvimagesink

Fails as we need to make sure the video is in a format that xvimagesink can understand, e.g. BGRx.

And with Nvidia plugins...
gst-launch-1.0 -vvv v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1 ! nvvidconv ! 'video/x-raw(memory:NVMM)' !  nvvidconv ! 'video/x-raw,format=GRAY8' !  nvvidconv ! nv3dsink -e


We can also do fun things like flip the image

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=160,height=120  ! nvvidconv flip-method=rotate-180 ! nv3dsink -e

Part3: Fun and games

some "special effects"

gst-launch-1.0 avfvideosrc device-index=0 ! videoconvert ! warptv ! videoconvert ! autovideosink

gst-launch-1.0 -v videotestsrc ! agingtv scratch-lines=15 ! videoconvert ! autovideosink

Some text: 

gst-launch-1.0 avfvideosrc device-index=0 ! videoconvert ! textoverlay text="Hello World" valignment=bottom halignment=left font-desc="Sans, 40" ! autovideosink

Fun with windows:
gst-launch-1.0 avfvideosrc device-index=0 ! queue ! tee name=t t. ! queue ! autovideosink sync=false t. ! queue ! videoflip method=horizontal-flip ! autovideosink sync=false -e

Picture in Picture...

gst-launch-1.0 nvcompositor name=mix sink_0::xpos=0 sink_0::ypos=0 sink_0::zorder=10 sink_1::xpos=0 sink_1::ypos=0 ! nvegltransform ! nveglglessink videotestsrc ! nvvidconv ! mix.sink_0 v4l2src device=/dev/video0 ! nvvidconv ! mix.sink_1

Now now encoding and decoding...

gst-launch-1.0 videotestsrc ! 'video/x-raw, format=(string)I420, width=(int)640, 
height=(int)480' ! omxh264enc ! 'video/x-h264, stream-format=(string)byte-stream' ! h264parse ! omxh264dec ! nveglglessink -e

(look at jtop)

Let's encode a file.


gst-launch-1.0 videotestsrc ! \
  'video/x-raw, format=(string)I420, width=(int)640, \
  height=(int)480' ! omxh264enc ! \
  'video/x-h264, stream-format=(string)byte-stream' ! h264parse ! \
  qtmux ! filesink location=test.mp4 -e

  Or with your video...

  gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1 ! nvvidconv ! omxh264enc ! 'video/x-h264, stream-format=(string)byte-stream' ! h264parse ! qtmux ! filesink location=test.mp4 -e


 and how to play the file back....

 gst-launch-1.0 filesrc location=test.mp4 ! qtdemux ! queue ! h264parse ! nvv4l2decoder ! nv3dsink -e


And from within Python and OpenCV

import numpy as np
import cv2

# use gstreamer for video directly; set the fps
camSet='v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1 ! videoconvert ! video/x-raw, format=BGR ! appsink'
cap= cv2.VideoCapture(camSet)

#cap = cv2.VideoCapture(0)

while(True):
    # Capture frame-by-frame
    ret, frame = cap.read()

    # Display the resulting frame
    cv2.imshow('frame',frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# When everything done, release the capture
cap.release()
cv2.destroyAllWindows()










----------------------------------------------
GStreamer Introduction and Demo
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
  
  
  

