Introduction
=========================================

Intel Low Precision Optimization Tool is a opensource python library to help users to fast deploy low-precision inference solution on popular DL frameworks including TensorFlow, PyTorch, MxNet etc. It is accuracy-driven objective tuning in tuning space and support different strategies.

# Infrastructure

<div align="left">
  <img src="imgs/infrastructure.jpg" width="700px" />
</div>

### Three level APIs

1. User API

   The user API are intented to provide best out-of-box experients and unify the low precision quantization interface cross differnt DL frameworks.

   ```
   def tune(self, model, q_dataloader, q_func=None,
            eval_dataloader=None, eval_func=None, resume_file=None)
   ```

   The tuning config and model-specific information are controlled by user config yaml file. As for the format of yaml file, please refer to [template.yaml](../examples/template.yaml)

   Intel® Low Precision Optimization Tool v1.0a release supports two usages:

   a) User specifies fp32 "model", calibration dataset "q_dataloader", evaluation dataset "eval_dataloader" and metric in tuning.metric field of model-specific yaml config file.

   b) User specifies fp32 "model", calibration dataset "q_dataloader" and a custom "eval_func" which encapsulates the evaluation dataset and metric by itself.

   The first usage is designed for seamless integrating user code with Intel® Low Precision Optimization Tool as well as the metric is supported by Intel® Low Precision Optimization Tool. Usually classfication networks are suitable for this.

   The second usage is designed for integrating those networks using Intel® Low Precision Optimization Tool not supported metrics with Intel® Low Precision Optimization Tool. Usually object detection and NLP networks are suitable for this.

2. Framework Adaption API

   Framework adaption layer is used to execute specific framework operations, such as calibration, quantization, evaluation and inspect tensor.

3. Extension API

   Intel® Low Precision Optimization Tool is easy to add new strategies, framework adaption layers, metrics and objectives easily by well-designed API.

   To add a new framework adaption layer, user just need inherit "Adaptor" class and implement a subclass under adaptor directory.

   To add a new strategy, user just need inherit "Strategy" class and implement a subclass under strategy directory.

   To add a new metric, user just need inherit "Metric" class and implement a subclass in metric.py.

   To add a new objective, user just need inherit "Objective" class and implement a subclass in objective.py.

# Workflow

<div align="left">
  <img src="imgs/workflow.jpg" width="700px" />
</div>

# Strategies

### Basic Strategy

This strategy is Intel® Low Precision Optimization Tool default tuning strategy, which does model-wise tuning by adjusting gloabl tuning parameters, such as calibration related parameters, kl or minmax algo, quantization related parameters, symmetric or asymmetric, per_channel or per_tensor. If the model-wise tuning result does not meet accuracy goal, this strategy will attempt to do op-wise fallback from bottom to top to prioritize which fallback op has biggest impact on final accuracy, and then do incremental fallback till achieving the accuracy goal.

### Bayersian Strategy

Bayesian optimization is a sequential design strategy for global optimization of black-box functions. The strategy refers to the Bayesian optimization package [bayesian-optimization](https://github.com/fmfn/BayesianOptimization) and changes it to a discrete version that complies with the Intel® Low Precision Optimization Tool strategy standard. It uses Gaussian Processes to define the prior/posterior distribution over the black-box function, and then finds the tuning config that maximizes the expected improvement.

### MSE Strategy

This strategy is very similar to the basic strategy. It needs to get the tensors for each Operator of raw FP32 models and the quantized model based on best model-wise tuning config. And then calculate the MSE (Mean Squared Error) for each operator, sort those operators according to the MSE value, finally do the op-wise fallback in this order.

# Objectives

Intel® Low Precision Optimization Tool supports below 3 build-in objectives:

### 1. Performance

This objective is used to measure performance of quantized model with accuracy driven. It is default objective Intel® Low Precision Optimization Tool supported.

### 2. Memory Footprint

This objective is used to measure the memory usage of evaluating quantized model with accuracy driven.

### 3. Model Size

This objective is used to measure the memory size of quantized model with accuracy driven.

# Metrics

Intel® Low Precision Optimization Tool supports 3 built-in metrics, Topk, F1 and CocoMAP. The metric is easily extensible as well as contributor implements a subclass of Metric class.
