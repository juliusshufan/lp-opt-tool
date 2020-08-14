Step-by-Step
============

This document is used to list steps of reproducing MXNet SSD-Mobilenet1.0/SSD-ResNet50_v1 with COCO dataset tuning zoo result.



# Prerequisite

### 1. Installation

  ```Shell
  # Install Intel® Low Precision Optimization Tool
  pip install ilit

  # Install MXNet
  pip install mxnet-mkl==1.6.0
  
  # Install gluoncv
  pip install gluoncv

  # Install pycocotool
  pip install pycocotools

  ```

### 2. Prepare Dataset

  Download COCO Raw image to work dir, naming as ~/.mxnet/dataset/coco.


# Run

### SSD-Mobilenet1.0

```bash
python eval_ssd.py --network=mobilenet1.0 --data-shape=512 --batch-size=256 --dataset coco
```

### SSD-ResNet50_v1
```bash
python eval_ssd.py --network=resnet50_v1 --data-shape=512 --batch-size=256 --dataset coco
```

Examples of enabling Intel® Low Precision Optimization Tool auto tuning on MXNet Object detection
=======================================================

This is a tutorial of how to enable a MXNet Object detection model with Intel® Low Precision Optimization Tool.

# User Code Analysis

Intel® Low Precision Optimization Tool supports two usages:

1. User specifies fp32 "model", calibration dataset "q_dataloader", evaluation dataset "eval_dataloader" and metric in tuning.metric field of model-specific yaml config file.

2. User specifies fp32 "model", calibration dataset "q_dataloader" and a custom "eval_func" which encapsulates the evaluation dataset and metric by itself.

As this example use COCO dataset, use COCOEval as metric which is can find [here](https://cocodataset.org/). So we integrate MXNet SSD-Mobilenet1.0/SSD-ResNet50_v1 with Intel® Low Precision Optimization Tool by the second use case.

### Write Yaml config file

In examples directory, there is a template.yaml. We could remove most of items and only keep mandotory item for tuning. 


```
#conf.yaml

framework:
  - name: mxnet

tuning:
    accuracy_criterion:
      - relative: 0.01
    timeout: 0
    random_seed: 9527
```

Because we use the second use case which need user to provide a custom "eval_func" encapsulates the evaluation dataset and metric, so we can not see a metric at config file tuning filed. We set accuracy target as tolerating 0.01 relative accuracy loss of baseline. The default tuning strategy is basic strategy. The timeout 0 means early stop as well as a tuning config meet accuracy target.


### code update

First, we need to construct evaluate function for ilit. At eval_func, we get the val_dataset for the origin script, and return mAP metric to ilit.

```python
    # define test_func
    def eval_func(graph):
        val_dataset, val_metric = get_dataset(args.dataset, args.data_shape)
        val_data = get_dataloader(
        val_dataset, args.data_shape, args.batch_size, args.num_workers)
        classes = val_dataset.classes  # class names

        size = len(val_dataset)
        ctx = [mx.cpu()]
        results = validate(graph, val_data, ctx, classes, size, val_metric)

        mAP = float(results[-1][-1])

        return mAP
```

After prepare step is done, we just need update main.py like below.

```python
   
    # Doing Intel® Low Precision Optimization Tool auto-tuning here
    import ilit
    ssd_tuner = ilit.Tuner("./ssd.yaml")
    ssd_tuner.tune(net, q_dataloader=val_data, eval_dataloader=val_dataset, eval_func=eval_func)
```

The tune() function will return a best quantized model during timeout constrain.
