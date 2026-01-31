---
title: "Types of Parallelism for LLM Inference or Deep Learning Explained"
date: 2026-01-31
draft: true
featured: false
---
TLDR; The parallelism you chose (the way you load your matrices on GPUs) can make the difference between utilization of 50% and 90%.

## Introduction

If you have ever used `vllm` you probably came across terms like *tensor parallel inference* or *pipeline parallel inference* terms [in the vllm documentation](https://docs.vllm.ai/en/stable/serving/parallelism_scaling/). The reason you decide for one over the other is the number of GPUs and the amount of available VRAM on each GPU.

The rule of thumb is:
> 1. If you have only one GPU and the model fits perfectly on one GPU, just use the GPU!

> 2. If you have 1 node and multiple GPUs and...

        a. The model fits on 1 GPU and you want to maximize throughput by using all the GPUS, use the *data Parallelism*.

        b. The model does NOT fit on a single GPU, use *tensor Parallelism*.

> 3. if the model is too large for a single node, combine *tensor parallelism* with *pipeline parallelism*.



## Types of Parallelism


## 1. Pipeline Parallelism

* pipeline bubbles

## 2. Tensor Parallelism


## 3. Data Parallelism

* talk about load balancer and it's importance


---

## My Personal Experience with these Concepts

I wanted to generate 4,000 data entries to compare with golden answers from GPT-OSS-120B model. I was working on a single node with 4xA40 NVIDIA GPUs and running [Llama-3.2-3B-Instruct](https://huggingface.co/meta-llama/Llama-3.2-3B-Instruct). I wanted to maximize the output throughput to utilize the GPUs to the best way possible. This was on an HPC environment provided by [NHR@FAU](https://hpc.fau.de/).

[to be continued...]

---

## My mistake when running the inference on 4 GPUs
I should've had used data parallel instead of tensor parallel since 1. the model would perfectly fit on one GPU and 2. Tensor parallelism has overhead of breaking and combining the calculation between 4 GPUs

## Let's Walk Through one Example

* Explain what happens in such a scenario:  `For example, set tensor_parallel_size=8 and pipeline_parallel_size=2 when using 2 nodes with 8 GPUs per node.` [example](https://docs.vllm.ai/en/stable/serving/parallelism_scaling/#distributed-inference-strategies-for-a-single-model-replica:~:text=For%20example%2C%20set%20tensor_parallel_size%3D8%20and%20pipeline_parallel_size%3D2%20when%20using%202%20nodes%20with%208%20GPUs%20per%20node.)



## Lessons learned


### EDITORIAL NOTES
[this section will be changed during drafting the post and will be removed in the end]
* include pictures from this paper https://xzt102.github.io/publications/2021_WWW.pdf



---

> I am open for collaboration. I am interested in solving and talking about problems related to serving AI and the system operations around it. If you have problems related to reliable deployment of AI systems (on cloud, HPC or even on-perm), feel free to write me an email.
