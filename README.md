# Nvidia TX2-Streaming
## H264 UDP Streaming
### Installing necessary modules
```
sudo apt-get install gstreamer1.0-tools gstreamer1.0-alsa gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav 
```
### H264 Encoding (TX2 board)
```
CLIENT_IP=192.168.1.101
gst-launch-1.0 nvcamerasrc fpsRange="30 30" intent=3 ! nvvidconv flip-method=6 \
! 'video/x-raw(memory:NVMM), width=(int)1920, height=(int)1080, format=(string)I420, framerate=(fraction)30/1' ! \
omxh264enc control-rate=2 bitrate=4000000 ! 'video/x-h264, stream-format=(string)byte-stream' ! \
h264parse ! rtph264pay mtu=1400 ! udpsink host=$CLIENT_IP port=5000 sync=false async=false
```

```
CLIENT_IP=192.168.1.101
gst-launch-1.0 nvarguscamerasrc ! nvvidconv flip-method=6 \
! 'video/x-raw(memory:NVMM), width=(int)1920, height=(int)1080, format=(string)I420, framerate=(fraction)30/1' ! \
omxh264enc control-rate=2 bitrate=4000000 ! 'video/x-h264, stream-format=(string)byte-stream' ! \
h264parse ! rtph264pay mtu=1400 ! udpsink host=$CLIENT_IP port=5000 sync=false async=false
```


```
python2 -c 'import sys; print (sys.path)'
python3 -c 'import sys; print (sys.path)'
```

### H264 Decoding
```
gst-launch-1.0 udpsrc port=5000 ! application/x-rtp,encoding-name=H264,payload=96 ! rtph264depay ! h264parse ! queue ! avdec_h264 ! xvimagesink sync=false async=false -e
```

## H265 UDP Streaming Decoding
```
gst-launch-1.0 udpsrc port=5000 ! application/x-rtp,encoding-name=H265,payload=96 ! rtph265depay ! h265parse ! queue ! avdec_h265 ! xvimagesink sync=false async=false -e
```

## H265 multicast decoding
```
gst-launch-1.0 udpsrc multicast-group=224.1.1.1 auto-multicast=true port=5000 ! application/x-rtp,encoding-name=H265,payload=96 ! rtph265depay ! h265parse ! queue ! avdec_h265 ! xvimagesink sync=false async=false -e
```
http://elinux.org/Jetson/Tutorials/Video_Stabilization

## GStreaming support opencv
From:
http://www.samontab.com/web/2017/06/installing-opencv-3-2-0-with-contrib-modules-in-ubuntu-16-04-lts/

In order to recompile opencv:

Note cmake is with additional -D WITH_CUDA=ON

```
cmake -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D WITH_V4L=ON -D INSTALL_C_EXAMPLES=ON -D INSTALL_PYTHON_EXAMPLES=ON -D WITH_OPENNI2=ON -D BUILD_EXAMPLES=ON -D WITH_QT=ON -D WITH_OPENGL=ON -D WITH_VTK=ON -D WITH_CUDA=ON -D ENABLE_FAST_MATH=1 -D CUDA_FAST_MATH=1 -D WITH_CUBLAS=ON -D CMAKE_BUILD_TYPE=RELEASE -D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-8.0/ -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-3.2.0/modules ..  &>out.txt
```
### Programming in opencv
```
import cv2

#cap = cv2.VideoCapture("rtsp://admin:xxxx@10.168.1.248:554/h264/ch1/main/av_stream")
cap = cv2.VideoCapture(0)

out = cv2.VideoWriter("appsrc ! video/x-raw, format=BGR ! queue ! videoconvert ! video/x-raw, format=BGRx ! nvvidconv ! omxh264enc ! video/x-h264, stream-format=byte-stream ! h264parse ! rtph264pay pt=96 config-interval=1 ! udpsink host=10.168.1.177 port=50001", cv2.CAP_GSTREAMER, 0, 25.0, (1920,1080))

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        # processed your frame here using yolov5, draw the bounding box onto frame
        if out.isOpened():
            out.write(frame)
            print('writing frame')
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
    else:
        break

# Release everything if job is finished
cap.release()
out.release()
```
## Jetson Camera Module

https://www.arducam.com/docs/camera-for-jetson-nano/native-jetson-cameras-imx219-imx477/imx477/
