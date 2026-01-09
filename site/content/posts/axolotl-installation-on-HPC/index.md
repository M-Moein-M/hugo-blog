---
title: "How to Install Axolotl on HPC environment"
date: 2026-01-04
draft: false
featured: true
---
TLDR; use containerization solutions instead of installing axolotl: docker or apptainer (for HPC)

## Introduction

You probably already know this but Axolotl (as documentation puts it) is...
> A Free and Open Source LLM Fine-tuning Framework

My main use case was to fine-tune [meta-llama/Llama-3.2-3B](https://huggingface.co/meta-llama/Llama-3.2-3B) on a synthesized/generate tool calling dataset. This was in a HPC environment provided by [NHR@FAU](https://hpc.fau.de/).

## Installation approaches

There are two main approaches for installing Axolotl on systems.

## 1. Containerization (highly recommended!)

### Apptainer on HPC

If you are working in HPC environment or a system that is managed externaly or shared among many users, it is most likely that there is no support for Docker. [Apptainer](https://apptainer.org/) is the containerazation solution on HPC and such environments. 

Below is the command that you can use to pull the axolot image (`.sif`) file and run it as a container.

```bash
mkdir axolotl && cd axolotl # or just change directory where you want the .sif file exist
apptainer pull axolotl.sif docker://axolotlai/axolotl:main-latest # pull the image
apptainer exec --nv axolotl.sif accelerate --help  # check if everything works
```

### Docker

If Docker is available on your system, you can easily pull the image according to the documentation instructions [here](https://docs.axolotl.ai/docs/installation.html#sec-docker).

## 2. pip/conda/uv (could not make it work)

Just to give you a heads up if you wanna go down this path, it took me 4 hours of trying searching and checking with 4 different AI assitants (ChatGPT, Groq, Gemini, Claude) and trying different installation orders (i.e. installing pytorch before flash-attn) and pinning different versions CUDA, python, and pytorch.

In my honest opinion you should not install Axolotl this way. In the future, I do my best to avoid installing such packages with python package managers and I will directly look for containerized solutions.

I allocated 1 GPU to make sure GPUS are available when installing (was hoping it would have some effects) and with 16 CPU cores available. Here is the CPU utilization during the installation of axolotl using `pip`. This should give you clear idea of how things will be if you try to install it with pip.

{{< figure src="./building-axolotl.png" alt="Building Axolotl" align="center" >}}

All the cores were fully busy and the installation ended up running into a build error.

## Lessons learned

For such complecated packages that are highly dependent on multiple big packages and CUDA version, it is almost mandatory to use these packages through their containerized version.

---

> I am interested in solving or talking about problems related to serving AI and the system operations around it. If you have problems related to reliable deployment of AI systems (on cloud, HPC or even on-perm) you can write me an email.