# Properties Guide

## Mandatory properties for classifiers

classifier-threshold, is-classifier

## Optional properties for classifiers

classifier-async-mode(Secondary mode only, Default=false)

## Optional properties in secondary mode

 operate-on-gie-id(Default=0), operate-on-class-ids(Defaults to all classes),
 input-object-min-width, input-object-min-height, input-object-max-width,
 input-object-max-height

## Following properties are always recommended

batch-size(Default=1)

## Other optional properties:

net-scale-factor(Default=1), network-mode(Default=0 i.e FP32),
model-color-format(Default=0 i.e. RGB) model-engine-file, labelfile-path,
mean-file, gie-unique-id(Default=0), offsets, process-mode (Default=1 i.e. primary),
custom-lib-path, network-mode(Default=0 i.e FP32) 

> The values in the config file are overridden by values set through GObject
 properties.