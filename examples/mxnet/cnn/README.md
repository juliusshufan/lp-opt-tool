Step-by-Step
============

This document is used to list steps of reproducing MXNet ResNet50_v1/Squeezenet1.0/MobileNet1.0/MobileNetv2_1.0/Inceptionv3 tuning zoo result.


# Prerequisite

### 1. Installation

  ```Shell
  # Install Intel® Low Precision Optimization Tool
  pip install ilit

  # Install MXNet
  pip install mxnet-mkl==1.6.0
  
  # Install gluoncv
  pip install gluoncv

  ```

### 2. Prepare Dataset

  Download validation dataset [here](http://data.mxnet.io/data/val_256_q90.rec), naming as /path/to/data/val_256_q90.rec.


# Run

### ResNet50_v1

```bash
python -u imagenet_inference.py \
        --symbol-file=./model/resnet50_v1-symbol.json \
        --param-file=./model/resnet50_v1-0000.params \
        --rgb-mean=123.68,116.779,103.939 \
        --rgb-std=58.393,57.12,57.375 \
        --batch-size=64 \
        --num-inference-batches=500 \
        --dataset=./data/val_256_q90.rec \
        --ctx=cpu 
```

#### Squeezenet1.0
```bash
python -u imagenet_inference.py \
        --symbol-file=./model/squeezenet1.0-symbol.json \
        --param-file=./model/squeezenet1.0-0000.params \
        --rgb-mean=123.68,116.779,103.939 \
        --rgb-std=58.393,57.12,57.375 \
        --batch-size=64 \
        --num-inference-batches=500 \
        --dataset=./data/val_256_q90.rec \
        --ctx=cpu 
```

### MobileNet1.0
```bash
python -u imagenet_inference.py \
        --symbol-file=./model/mobilenet1.0-symbol.json \
        --param-file=./model/mobilenet1.0-0000.params \
        --rgb-mean=123.68,116.779,103.939 \
        --rgb-std=58.393,57.12,57.375 \
        --batch-size=64 \
        --num-inference-batches=500 \
        --dataset=./data/val_256_q90.rec \
        --ctx=cpu 
```

### MobileNetv2_1.0
```bash
python -u imagenet_inference.py \
        --symbol-file=./model/mobilenetv2_1.0-symbol.json \
        --param-file=./model/mobilenetv2_1.0-0000.params \
        --rgb-mean=123.68,116.779,103.939 \
        --rgb-std=58.393,57.12,57.375 \
        --batch-size=64 \
        --num-inference-batches=500 \
        --dataset=./data/val_256_q90.rec \
        --ctx=cpu \
```

### Inceptionv3
```bash
python -u imagenet_inference.py \
        --symbol-file=./model/inceptionv3-symbol.json \
        --param-file=./model/inceptionv3-0000.params \
        --rgb-mean=123.68,116.779,103.939 \
        --rgb-std=58.393,57.12,57.375 \
        --batch-size=64 \
        --image-shape 3,299,299 \
        --num-inference-batches=500 \
        --dataset=./data/val_256_q90.rec \
        --ctx=cpu
```

Examples of enabling Intel® Low Precision Optimization Tool auto tuning on MXNet ResNet50
=======================================================

This is a tutorial of how to enable a MXNet classification model with Intel® Low Precision Optimization Tool.

# User Code Analysis

Intel® Low Precision Optimization Tool supports two usages:

1. User specifies fp32 "model", calibration dataset "q_dataloader", evaluation dataset "eval_dataloader" and metric in tuning.metric field of model-specific yaml config file.

2. User specifies fp32 "model", calibration dataset "q_dataloader" and a custom "eval_func" which encapsulates the evaluation dataset and metric by itself.

>As ResNet50_v1/Squeezenet1.0/MobileNet1.0/MobileNetv2_1.0/Inceptionv3 series are typical classification models, use Top-K as metric which is built-in supported by Intel® Low Precision Optimization Tool. So here we integrate MXNet ResNet with Intel® Low Precision Optimization Tool by the first use case for simplicity.

### Write Yaml config file

In examples directory, there is a template.yaml. We could remove most of items and only keep mandotory item for tuning. 


```
#conf.yaml

framework:
  - name: mxnet

tuning:
    metric:
      - topk: 1
    accuracy_criterion:
      - relative: 0.01
    timeout: 0
    random_seed: 9527
```

Here we choose topk built-in metric and set accuracy target as tolerating 0.01 relative accuracy loss of baseline. The default tuning strategy is basic strategy. The timeout 0 means early stop as well as a tuning config meet accuracy target.


### code update

After prepare step is done, we just need update imagenet_inference.py like below.

```python
    import ilit
    fp32_model = load_model(symbol_file, param_file, logger)
    cnn_tuner = Tuner("./cnn.yaml")
    cnn_tuner.tune(fp32_model, q_dataloader=data, eval_dataloader=data)

```

The tune() function will return a best quantized model during timeout constrain.
