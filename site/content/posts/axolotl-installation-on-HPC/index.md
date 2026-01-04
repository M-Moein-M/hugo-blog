---
title: "How to Install Axolotl on HPC environment"
date: 2026-01-04
draft: false
featured: true
---
TLDR; use containerization solutions instead of installing axolotl. docker or apptainer (for HPC)

## Introduction

You probably already know this but Axolotl (as documentation puts it) is...
> A Free and Open Source LLM Fine-tuning Framework

My main use case was to fine-tune [meta-llama/Llama-3.2-3B](https://huggingface.co/meta-llama/Llama-3.2-3B) on a synthesized/generate tool calling dataset. This was in a HPC environment provided by [NHR@FAU](https://hpc.fau.de/).

# installation possibilities/approaches

There are two main approaches for installing Axolotl on systems.

## 1. Containerization (what worked for me)

### apptainer on HPC

If you are working in HPC environment or a system that is managed with Slurm job manager, it is most likely that there is no support for Docker. Apptainer is the containerazation solution on HPC. 

Below is the command that you can use to pull the axolot image (`.sif`) file and run it as a container.

```bash
# TODO add the code for downloading and running axolotl apptainer 
```

### Docker

If Docker is available on your system, you can easily pull the image based on the documentation instructions [here](https://docs.axolotl.ai/docs/installation.html#sec-docker).

## 2. pip/conda/uv (painful approach)

Just to give you a heads up if you wanna go down this path, it took me 4 hours of trying searching and checking with 4 different AI assitants (ChatGPT, Groq, Gemini, Claude) and trying different installation orders (i.e. installing pytorch before flash-attn) and pinning different versions CUDA, python, and pytorch.

In my honest opinion you should not install Axolotl this way. In the future, I do my best to avoid installing such packages with python package managers and I will directly look for containerized solutions.


## binding resources
