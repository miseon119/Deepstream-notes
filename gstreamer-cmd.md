# GStreamer command samples

## Jetson CSI Camera

### Resize to (640, 360)
```console
$ gst-launch-1.0 nvarguscamerasrc sensor_id=0 ! 'video/x-raw(memory:NVMM),width=3264, height=2464, framerate=21/1, format=NV12' ! \
nvvidconv flip-method=0 ! 'video/x-raw, width=640, height=360' !    nvvidconv ! nvegltransform ! nveglglessink -e
```

### Save as out.mp4
```console
$ gst-launch-1.0 nvarguscamerasrc sensor-id=0 ! "video/x-raw(memory:NVMM), width=3264, height=2464, format=NV12, framerate=21/1" ! \
tee name=t ! queue ! videorate !  nvv4l2h264enc ! h264parse ! qtmux ! filesink location=out.mp4 t. ! queue ! \
nvoverlaysink sync=0 overlay-x=640 overlay-y=0 overlay-w=1280 overlay-h=720 -e
```

[Reference](https://developer.ridgerun.com/wiki/index.php?title=Gstreamer_pipelines_for_Jetson_TX2)
