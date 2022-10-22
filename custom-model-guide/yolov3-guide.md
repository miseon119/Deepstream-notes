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

[more properties](https://github.com/miseon119/Deepstream-notes/blob/main/custom-model-guide/nvinfer-properties-guide.md#mandatory-properties-for-classifiers)

## Set Engine Create Function Name
```
parse-bbox-func-name=NvDsInferParseCustomYoloV3
custom-lib-path=nvdsinfer_custom_impl_Yolo/libnvdsinfer_custom_impl_Yolo.so
engine-create-func-name=NvDsInferYoloCudaEngineGet
```

First, Write `NvDsInferYoloCudaEngineGet` function:
```cpp
#if !USE_CUDA_ENGINE_GET_API
IModelParser* NvDsInferCreateModelParser(
    const NvDsInferContextInitParams* initParams) {
    NetworkInfo networkInfo;
    if (!getYoloNetworkInfo(networkInfo, initParams)) {
      return nullptr;
    }

    return new Yolo(networkInfo);
}
#else
extern "C"
bool NvDsInferYoloCudaEngineGet(nvinfer1::IBuilder * const builder,
        nvinfer1::IBuilderConfig * const builderConfig,
        const NvDsInferContextInitParams * const initParams,
        nvinfer1::DataType dataType,
        nvinfer1::ICudaEngine *& cudaEngine);

extern "C"
bool NvDsInferYoloCudaEngineGet(nvinfer1::IBuilder * const builder,
        nvinfer1::IBuilderConfig * const builderConfig,
        const NvDsInferContextInitParams * const initParams,
        nvinfer1::DataType dataType,
        nvinfer1::ICudaEngine *& cudaEngine)
{
    NetworkInfo networkInfo;
    if (!getYoloNetworkInfo(networkInfo, initParams)) {
      return false;
    }

    Yolo yolo(networkInfo);
    cudaEngine = yolo.createEngine (builder, builderConfig);
    if (cudaEngine == nullptr)
    {
        std::cerr << "Failed to build cuda engine on "
                  << networkInfo.configFilePath << std::endl;
        return false;
    }

    return true;
}
#endif
```

## Set parse-bbox-func-name

```
parse-bbox-func-name=NvDsInferParseCustomYoloV3
```

```cpp
extern "C" bool NvDsInferParseCustomYoloV3(
    std::vector<NvDsInferLayerInfo> const& outputLayersInfo,
    NvDsInferNetworkInfo const& networkInfo,
    NvDsInferParseDetectionParams const& detectionParams,
    std::vector<NvDsInferParseObjectInfo>& objectList);

/* C-linkage to prevent name-mangling */
extern "C" bool NvDsInferParseCustomYoloV3(
    std::vector<NvDsInferLayerInfo> const& outputLayersInfo,
    NvDsInferNetworkInfo const& networkInfo,
    NvDsInferParseDetectionParams const& detectionParams,
    std::vector<NvDsInferParseObjectInfo>& objectList)
{
    static const std::vector<float> kANCHORS = {
        10.0, 13.0, 16.0,  30.0,  33.0, 23.0,  30.0,  61.0,  62.0,
        45.0, 59.0, 119.0, 116.0, 90.0, 156.0, 198.0, 373.0, 326.0};
    static const std::vector<std::vector<int>> kMASKS = {
        {6, 7, 8},
        {3, 4, 5},
        {0, 1, 2}};
    return NvDsInferParseYoloV3 (
        outputLayersInfo, networkInfo, detectionParams, objectList,
        kANCHORS, kMASKS);
}

```