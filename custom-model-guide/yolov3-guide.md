# Yolov3 Custom Library Guide

## Compile the custom library

1. Based on the API to use `NvDsInferCreateModelParser` or `NvDsInferCudaEngineGet` 
 set the macro `USE_CUDA_ENGINE_GET_API` to 0 or 1 in `nvdsinfer_custom_impl_Yolo/nvdsinfer_yolo_engine.cpp`

2. Export correct CUDA version as per the platform
 
  For Jetson:  $ export CUDA_VER=11.4
 
  For x86:     $ export CUDA_VER=11.6

```bash
sudo -E make -C nvdsinfer_custom_impl_Yolo
```

## nvinfer config file 

The "nvinfer" config file `config_infer_primary_yolo.txt` specifies the path to
the **custom library** and the **custom output parsing function** through the properties
`custom-lib-path` and `parse-bbox-func-name` respectively.

```
[property]
gpu-id=0
net-scale-factor=0.0039215697906911373
#0=RGB, 1=BGR
model-color-format=0
custom-network-config=yolov3.cfg
model-file=yolov3.weights
model-engine-file=model_b1_gpu0_fp16.engine
labelfile-path=labels.txt
int8-calib-file=yolov3-calibration.table.trt7.0
## 0=FP32, 1=INT8, 2=FP16 mode
network-mode=2
num-detected-classes=80
gie-unique-id=1
network-type=0
is-classifier=0
## 1=DBSCAN, 2=NMS, 3= DBSCAN+NMS Hybrid, 4 = None(No clustering)
cluster-mode=2
maintain-aspect-ratio=1
parse-bbox-func-name=NvDsInferParseCustomYoloV3
custom-lib-path=nvdsinfer_custom_impl_Yolo/libnvdsinfer_custom_impl_Yolo.so
engine-create-func-name=NvDsInferYoloCudaEngineGet
#scaling-filter=0
#scaling-compute-hw=0

[class-attrs-all]
nms-iou-threshold=0.3
threshold=0.7
```

Following properties are mandatory when engine files are **not specified**:
1.  int8-calib-file(Only in INT8), model-file-format
2.   Caffemodel mandatory properties: model-file, proto-file, output-blob-names
3.   UFF: uff-file, input-dims, uff-input-blob-name, output-blob-names
4.   ONNX: onnx-file


Mandatory properties for detectors:

`num-detected-classes`

Optional properties for detectors:

`cluster-mode`(Default=Group Rectangles), interval(Primary mode only, Default=0)

`custom-lib-path`

`parse-bbox-func-name`

[more properties](https://github.dev/miseon119/Deepstream-notes/blob/1b2293e029cf43bcf1fbc853c51688b7cd775922/custom-model-guide/nvinfer-properties-guide.md#L17)
