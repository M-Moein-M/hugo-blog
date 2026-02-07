---
title: "[Draft] Types of Parallelism for LLM Inference or Deep Learning Explained"
date: 2026-01-31
draft: false
featured: false
---
TLDR; The parallelism you chose (the way you load your matrices on GPUs) can make the difference between utilization of 50% and 90%.

## Introduction

If you have ever used `vllm` you probably came across terms like *tensor parallel inference* or *pipeline parallel inference* terms [in the vllm documentation](https://docs.vllm.ai/en/stable/serving/parallelism_scaling/). The reason you decide for one over the other is the number of GPUs and the amount of available VRAM on each GPU.

The rule of thumb is:
> 1. If you have only one GPU and the model fits perfectly on one GPU, just use the GPU!

> 2. If you have 1 node and multiple GPUs and...
> - The model fits on 1 GPU and you want to maximize throughput by using all the GPUS, use the *data Parallelism*.
> - The model does NOT fit on a single GPU, use *tensor Parallelism*.

> 3. if the model is too large for a single node, combine *tensor parallelism* with *pipeline parallelism*.



## Types of Parallelism

There are (at least) 3 main types of parallelism when it comes to serving LLM on GPUs. Basically the type of parallelism that you choose dictates how your model is distributed among your GPUs. Pictures below will hopefully give you good idea about the concept.

I also took the images from [this paper](https://xzt102.github.io/publications/2021_WWW.pdf)

## 1. Pipeline Parallelism


This approach splits the model by it's layers and each set of layers will live on different GPUs.

For example, if you have a Neural Network with 30 layers and 3 GPUs, this approach will put layers [0 to 9] on GPU number 1, layers [10 to 19] on GPU number 2 and so on.

{{< figure src="./pipeline.jpg" alt="Pipeline Parallelism Figure" align="center" >}}

### Pipeline Bubbles

One really important concept that you should be aware of is that when you put each layer on a different GPU, through out the computations, one layer should wait for the last layer to finish its calculations and pass the data. This adds small pauses during the computations from GPU to GPU and this is something known as Pipeline Bubbles.

## 2. Tensor Parallelism

This approach splits the matrices and puts them on different GPUs. So basically each GPU will hold one part of the each layer. 

For example, if you have a Neural Network with 30 layers of 2x2 matrices and 3 GPUs, the element at (0, 0) of each layer will sit in GPU number 1, the element at (0, 1) will sit in GPU number 2, and the elements of (1, 0) and (1, 1) of each layer will sit on GPU number 3.

{{< figure src="./tensor.jpg" alt="Tensor Parallelism Figure" align="center" >}}


## 3. Data Parallelism

In data parallelism the entire model sits on a single GPU and it is exactly copied among the other GPUs. This approach just replicates the model among the available hardware. 

{{< figure src="./data.jpg" alt="Pipeline Parallelism Figure" align="center" >}}

You might consider this approach when there are no hardware restrictions to fit the model on a single GPU but you probably only want to increase your throughput. For example if you have a model that fits on one GPU but you have 3 available GPUs, you can just use all three with data parallelism to multiply your throughput by a factor of 3 (best case).

This approach is a bit different than the other two. For pipeline and Tensor parallelism you cannot get *ANY* work done since your model does not fit on one GPU. So you use more hardware just to be able to use the model. But in data parallelism, you replicate the model and use more hardware to increase the throughput.

### Load Balancer

If all of your GPUs are holding one replica of the entire model, the question would be who should handle the next incoming request. This is something that is handled by a load balancer. The criteria on which the load balancer decides how to route the incoming request, can be things like Round Robin, load-aware (give the work to whichever that has less work to do) and more.

If you work with vllm, it comes with an [internal load balancer](https://docs.vllm.ai/en/stable/serving/data_parallel_deployment/#internal-load-balancing) to expose a single API endpoint when you are using Data Parallelism.


---

## My Personal Experience with these Concepts

I wanted to generate 4,000 data entries to compare with golden answers from GPT-OSS-120B model. I was working on a single node with 4xA40 NVIDIA GPUs and running [Llama-3.2-3B-Instruct](https://huggingface.co/meta-llama/Llama-3.2-3B-Instruct). I wanted to maximize the output throughput to utilize the GPUs to the best way possible. This was on an HPC environment provided by [NHR@FAU](https://hpc.fau.de/).


### Parallelism in Action

I was running *Llama-3.2-3B-Instruct* on 4xA40 NVIDIA GPUs using pipeline parallelism. But the utilization was way off that what I was expecting and the reason was that I was simply using the wrong parallelism approach. Below is a figure that shows the utilization of the GPUs during the inference (data generation).

{{< figure src="./gpu_util_pipeline.png" alt="GPU Utilization with Pipeline Parallelism" align="center" >}}

I changed parallelism approach from pipeline to tensor parallelism and the GPU utilization went up to 90% and stayed there!

{{< figure src="./tensor-utilization.jpg" alt="GPU Utilization with Tensor Parallelism" align="center" >}}


### \[IMPORTANT\] My mistake when running the inference on 4 GPUs
One thing I realized when I was writing this blog post is that I should have had used data parallel instead of tensor parallel since...
 1. the model would perfectly fit on one GPU
 2. Tensor parallelism has GPU communication overhead

And I could simply ~4X the through put by using data parallelism approach.

---

## Let's Walk Through one Example

Let's say you set `tensor_parallel_size=8` and `pipeline_parallel_size=2` and you are using 2 nodes each with 8 GPUs.

> Would first the tensor parallelism be applied and then pipeline parallelism or vice versa?

First we break the model layer-wise. We put each sub-layers on each node, and within that node, we break the tensors among the available GPUs. For example, If you have a model with 32 layers, the first 8 layers will sit on node-1 and the second 8 layers will sit on node-2. The first 8 layers, will be distributed via tensor parallel approach among all the available GPUs on that node.

I took the image below from [this blog post](https://tj-solergibert.github.io/post/3d-parallelism)

{{< figure src="./multinode.png" alt="Multi-node parallelism" align="center" >}}


## Lessons learned

\[TODO\]

---

> I am open for collaboration. I am interested in solving and talking about problems related to serving AI and the system operations around it. If you have problems related to reliable deployment of AI systems (on cloud, HPC or even on-perm), feel free to write me an email.
