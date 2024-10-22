---
description: Learn the basics of distributed training on Lambda Cloud
tags:
  - 1-click clusters
  - automation
  - on-demand cloud
---

# Tutorial: Getting started with training a machine learning model

## Overview

This tutorial walks you through the process of getting ready to start training a model using Lambda solutions. It helps you to scope out and decide what resources you are going to need and which Lambda solution to use.

This tutorial is aimed at machine learning engineers and systems administrators responsible for setting up the training environment.

## Before you begin

Before you make your Lambda reservation, it’s a good idea to scale out your project so you know which path to take.&#x20;

### Scope your project

Start by getting a sense of the overall scope and strategy of your project. Some factors to consider include:

* **Batch size**: Are you processing a single stream or multiple streams? Processing eight parallel streams at the same time is much different than processing a single stream.
* **Model size and composition**: How big is your model? How many copies of your model can you fit on a single GPU, if any? What are the main components of the model? A transformer scales differently with input length than a pure convolutional model.
* **Time**: If you have time constraints, can you quantize the model to a lower precision to finish training faster? Or can you distill the model so you can train a smaller one?
* **Data type and size**: Is your training data composed of text, images, or something else? Is it a recurring process, like a stream of text? Or do you plan to have all the input processed at once, like with a dataset of documents?

Once you have a general idea of the scope, you can move on to determine how much of each resource (GPUs/CPUs, VRAM, and storage) you need.

### VRAM

The amount of VRAM on a GPU helps determine how large your dataset and model can be without needing to offload to the system memory, which decreases efficiency.&#x20;

Use the following formula to estimate how much VRAM you need:

```bash
Memory required ≈
(model size x precision) +
(batch size x sequence length x input dimension x precision) +
(optimizer state x precision)
```

Where:

* **Model size**: is the number of parameters in your model
* **Precision**: use 4 bytes for single precision (FP32) or 2 bytes for mixed-precision training
* **Batch size**: is the number of samples to process in parallel
* **Sequence length**: is the length of the input sequences in bytes
* **Input dimension**: is the dimensionality of the input data; for example, use 128 for 128-dimensional word embeddings
* **Optimizer state**: is the size of the optimizer's internal state, which depends on the specific optimizer

Lambda also offers its own [VRAM calculator](https://huggingface.co/spaces/lambdabrendan/Lambda-LLM-Calculator) for large language models.

### GPUs and CPUs

Once you know how much VRAM you need, you can then determine the minimum number of GPUs you need, since the number of GPUs is bound by the memory requirements and the memory limitations of a given GPU. You can then scale out to see how many more GPUs you can use, since utilizing more GPUs for your training speeds it up, reducing training time, in a nearly linear fashion.&#x20;

If you are training an LLM, the Lambda [VRAM calculator](https://huggingface.co/spaces/lambdabrendan/Lambda-LLM-Calculator) can tell you the minimum number of GPUs you need.&#x20;

While GPUs occupy most of the spotlight when it comes to their importance in training ML models, CPUs play a very important role as well. Their workloads are more varied and fall into these categories:

* ML operations or workload management, including the monitoring stack
* Dedicated CPU processing tasks, typically in the pre- and post-processing of data sets
* Feeding the GPUs a consistent stream of data before the training application can take advantage of direct memory access (DMA) between GPUs or the storage devices

Lambda provides sufficient CPU resources for the GPUs you can select when you make a reservation.

### Storage

To estimate how much storage you need, follow this rule of thumb: take the number of parameters you plan to train on and multiply that number by 2 bytes, as each parameter typically requires 2 bytes for 16-bit precision. For example, a 70 billion parameter model would require roughly 140GB of disk space.

For 32-bit precision, double that number to 4 bytes per parameter. So this same 70 billion parameter model would require at least 280GB of storage.

That said, training data size can vary drastically between domains, depending on whether you are training on NLP, LLM, vision, or audio data. In general, datasets typically require hundreds of gigabytes of storage, so having 1TB or 2TB in total provides a comfortable cushion.

### Container or virtual environment

A container or virtual environment can help streamline your AI training process by offering a consistent, isolated environment, better resource management, and easier collaboration.&#x20;

Lambda recommends these containers and virtual environments:

* **Docker**: a Docker container includes the whole runtime environment so you can train your model in the exact same environment on any cloud.
* **Conda**: Conda provides a virtual environment that is isolated from the system it runs on, so you can train your model consistently on any system while avoiding conflicts with the host system.
* **Python**: Python is a programming language with a virtual environment module, `venv`, that you use to create virtual environments.

For more information on creating the container or virtual environment, read [Virtual environments and Docker containers](programming/virtual-environments-containers.md).

### On-Demand Cloud, 1-Click Cluster, or Private Cloud

Now that you have a good idea of what resources you need, you can choose the best product to suit your needs.

* **On-Demand Cloud**: is great for smaller models and shorter durations. On-Demand Cloud instances offer a choice from one to eight GPUs that you can rent by the hour.
* **1-Click Clusters**: is best for larger models requiring 16 to 512 GPUs that you can rent by the week, with discounts for longer terms.
* **Private Cloud**: is aimed at the largest and longest-lasting deployments. Lambda Private Cloud provides you with a cluster of 512 to 2048 GPUs for one to three years.

If you need to start working immediately, choose On-Demand Cloud or reserve a 1-Click Cluster for the near term. If you are on a longer timeline and intend to train your AI over a period of years using a cluster of GPUs, then you should choose Lambda Private Cloud.

This tutorial does not cover Private Cloud. For more information about these options and for pricing, visit the [Lambda website](https://lambdalabs.com/service/gpu-cloud).

## Launch a Lambda instance

Before you can launch a Lambda On-Demand Cloud instance or 1-Click Cluster, you need to [create a Lambda Public Cloud account](https://cloud.lambdalabs.com/sign-up) if you have not already done so and provide the following information:

* Account type (individual or business)
* Your contact information
* Your billing information

Agree to Lambda’s terms and conditions, go to your [Lambda Cloud dashboard](https://cloud.lambdalabs.com/instances) in your browser, and then launch an instance:

* To launch an On-Demand Cloud instance, click **Launch instance** and follow the steps under [Launch instances](https://docs.lambdalabs.com/on-demand-cloud/dashboard#launch-instances).
* To launch a 1-Click Cluster, click **Request 1-Click Cluster** then follow the steps under [Reserving 1-Click Clusters](https://docs.lambdalabs.com/1-click-clusters/getting-started#reserving-1-click-clusters).

You also need to generate an SSH key, which you select during the launch process. Read the section on [adding or generating an SSH key](https://docs.lambdalabs.com/on-demand-cloud/dashboard#add-generate-and-delete-ssh-keys) in the Dashboard chapter.&#x20;

### Lambda Stack

Every Lambda instance comes with the Lambda Stack preinstalled. The Lambda Stack is an AI software suite that includes PyTorch, TensorFlow, CUDA, cuDNN, and NVIDIA drivers. It runs on Ubuntu 22.04 LTS and 20.04 LTS. For more information, read [our website](https://lambdalabs.com/lambda-stack-deep-learning-software#server-installation) and the [overview page](https://docs.lambdalabs.com/on-demand-cloud/overview#preinstalled-software).&#x20;

### Configure the container or virtual environment

With your instance or cluster up and running, you can create the Docker container or Conda or Python virtual environment in which you will run your training model. For more information, read [Virtual environments and Docker containers](programming/virtual-environments-containers.md).

Once the container is running, check out your code repository to the container or virtual environment. If you don’t have a testing repository yet, you can try one of the many examples suggested below.

## Next steps

The final step before you start training is to choose a training model.

If you need guidance on where to go next, Lambda offers many different examples that you can run in your instance or cluster and see how they perform. &#x20;

* The [Lambda distributed training guide](https://github.com/LambdaLabsML/distributed-training-guide/tree/main) is a comprehensive introduction to training, covering a range of topics from a single GPU to a cluster. Clone the relevant chapter into your instance or cluster and give it a try. Note that these examples run in a [Python virtual environment](https://docs.lambdalabs.com/software/virtual-environments-and-docker-containers#creating-a-python-virtual-environment).&#x20;
  * [A single GPU on a single node](https://github.com/LambdaLabsML/distributed-training-guide/tree/main/01-single-gpu). This is the most basic example. Since it is limited to one GPU on one node, there is no distributed training involved.
  * [Multiple GPUs on a single node](https://github.com/LambdaLabsML/distributed-training-guide/tree/main/02-multi-gpu). This example is more involved and actually demonstrates distributed training, since it splits the workload — the batches from the dataset — over multiple GPUs.
  * [Multiple GPUs across multiple nodes](https://github.com/LambdaLabsML/distributed-training-guide/tree/main/03-multi-node). You need a multi-node instance to try this example. It demonstrates the need for identical environments.
  * GPU cluster. In order to test across a cluster, you need a mechanism to launch the training jobs on each node. You can use [Bash](https://github.com/LambdaLabsML/distributed-training-guide/tree/main/04-job-launchers-bash), [DeepSpeed](https://github.com/LambdaLabsML/distributed-training-guide/tree/main/04-job-launchers-deepspeed), [`mpirun`](https://github.com/LambdaLabsML/distributed-training-guide/tree/main/04-job-launchers-mpirun), or [Slurm](https://github.com/LambdaLabsML/distributed-training-guide/tree/main/04-job-launchers-slurm) as the launcher.
* For more targeted examples, try these [deep learning tutorials](https://github.com/LambdaLabsML/examples):
  * [Fine tuning Stable Diffusion](https://github.com/LambdaLabsML/examples/tree/main/stable-diffusion-finetuning). This example runs through performing extra training on a model to get more desirable results.
  * [Object detection using YoloV5](https://github.com/LambdaLabsML/examples/tree/main/yolov5). This workflow downloads a dataset, prepares it for YoloV5, then downloads pretrained weights files.
  * [GPTNeoX LLM distributed training](https://github.com/LambdaLabsML/examples/tree/main/gpt-neox-training). This example shows how to train a LLM across multiple nodes on Lambda Cloud. It uses the open source `gpt-neox` repository, which is built on DeepSpeed and MegatronLM.
  * [Experiment tracking comparison](https://github.com/LambdaLabsML/examples/tree/main/experiment-tracking). This example compares these common experiment tracking libraries: Comet, MLflow, Neptune.ai, and `wandb`.
  * [PyTorch multi-node training](https://github.com/LambdaLabsML/examples/tree/main/pytorch/distributed). This example shows how to write and launch PyTorch distributed data parallel jobs. We walk you through this example on the [Lambda blog](https://lambdalabs.com/blog/multi-node-pytorch-distributed-training-guide)
  * [Running Nerfstudio on Lambda](https://github.com/LambdaLabsML/examples/tree/main/nerfstudio). This example demonstrates how to create and view NeRF (Neural Radiance Fields) models.

\
\
\
