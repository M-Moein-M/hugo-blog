---
title: "How to Install Axolotl on HPC environment"
date: 2026-01-04
draft: false
featured: false
---
TLDR; use containerization solutions instead of installing axolotl: docker or apptainer (for HPC)

{{< youtube sQLQTiFf72U >}}

## Introduction

You probably already know this but Axolotl (as documentation puts it) is...
> A Free and Open Source LLM Fine-tuning Framework

My main use case was to fine-tune [Llama-3.2-3B-Instruct](https://huggingface.co/meta-llama/Llama-3.2-3B-Instruct) on a synthesized/generate tool calling dataset. This was in an HPC environment provided by [NHR@FAU](https://hpc.fau.de/).

## Installation approaches

There are two main approaches for installing Axolotl on systems.

## 1. Containerization (highly recommended!)

### Apptainer on HPC

If you are working in HPC environment or a system that is managed externally or shared among many users, it is most likely that there is no support for Docker. [Apptainer](https://apptainer.org/) is the containerization solution on HPC and such environments. 

Below is the command that you can use to pull the axolot image (`.sif`) file and run it as a container.

```bash
mkdir axolotl && cd axolotl # or just change directory where you want the .sif file exist
apptainer pull axolotl.sif docker://axolotlai/axolotl:main-latest # pull the image
apptainer exec --nv axolotl.sif accelerate --help  # check if everything works
apptainer cache clean  # for me running this released ~40GB of disk space
```

### Docker

If Docker is available on your system, you can easily pull the image according to the documentation instructions [here](https://docs.axolotl.ai/docs/installation.html#sec-docker).

## 2. pip/conda/uv (could not make it work)

Just to give you a heads up if you wanna go down this path, it took me 4 hours of trying searching and checking with 4 different AI assistants (ChatGPT, Groq, Gemini, Claude) and trying different installation orders (i.e. installing pytorch before flash-attn) and pinning different CUDA versions, python, and pytorch.

In my honest opinion you should not install Axolotl this way. In the future, I do my best to avoid installing such packages with python package managers and I will directly look for containerized solutions.

I allocated 1 GPU to make sure GPUS are available when installing (was hoping it would have some effects) and with 16 CPU cores available. Here is the CPU utilization during the installation of axolotl using `pip`. This should give you clear idea of how things will be if you try to install it with pip.

{{< figure src="./building-axolotl.png" alt="Building Axolotl" caption="CPU utilization during Axolotl installation using pip" align="center" >}}

All the cores were fully busy and the installation ended up running into a build error.

## VLLM example

This is a definition file that create a .sif which you can use to serve LLM.

Create a `vllm.def` file and copy the content below inside.

```bash
Bootstrap: docker
# From: nvcr.io/nvidia/cuda:12.8.1-cudnn-devel-ubuntu22.04
From: nvcr.io/nvidia/cuda:12.8.1-cudnn-runtime-ubuntu22.04

%labels
    Author your-name
    Version 1.0
    Description vLLM container with CUDA 12.8.1

%environment
    export PATH=/opt/venv/bin:$PATH
    export PYTHONPATH=/opt/venv/lib/python3.11/site-packages:$PYTHONPATH

    # CUDA / GPU environment
    export CUDA_HOME=/usr/local/cuda
    export PATH=$CUDA_HOME/bin:$PATH
    export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH

    # vLLM tuning knobs (adjust as needed)
    export VLLM_WORKER_MULTIPROC_METHOD=spawn
    export NCCL_DEBUG=WARN

%post
    set -e

    # ── System packages ────────────────────────────────────────────────────────
    apt-get update -y && apt-get install -y --no-install-recommends \
        python3.11 \
        python3.11-dev \
        python3.11-venv \
        python3-pip \
        git \
        curl \
        wget \
        build-essential \
        libssl-dev \
        libffi-dev \
        libnuma-dev \
        && apt-get clean && rm -rf /var/lib/apt/lists/*

    # Make python3.11 the default
    update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1
    update-alternatives --install /usr/bin/python  python  /usr/bin/python3.11 1

    # ── Virtual environment ────────────────────────────────────────────────────
    python3 -m venv /opt/venv
    . /opt/venv/bin/activate

    pip install --upgrade pip setuptools wheel

    # ── PyTorch (CUDA 12.8 / cu128 wheel) ─────────────────────────────────────
    pip install torch torchvision torchaudio \
        --index-url https://download.pytorch.org/whl/cu128

    # ── vLLM ──────────────────────────────────────────────────────────────────
    # Pin a specific release for reproducibility; bump as needed.
    pip install vllm==0.13.0

    # ── Optional: common serving / monitoring extras ───────────────────────────
    pip install \
        accelerate \
        transformers \
        huggingface_hub \
        sentencepiece \
        protobuf \
        ray

    # Cleanup
    pip cache purge

%runscript
    #!/bin/bash
    # Default: launch the vLLM OpenAI-compatible server.
    # Override by passing arguments:
    #   apptainer run vllm.sif python -m vllm.entrypoints.openai.api_server ...
    exec python -m vllm.entrypoints.openai.api_server "$@"

%test
    . /opt/venv/bin/activate
    python -c "import torch; print('PyTorch:', torch.__version__)"
    python -c "import torch; print('CUDA available:', torch.cuda.is_available())"
    python -c "import vllm; print('vLLM:', vllm.__version__)"
```

You can set a cache directory (which I highly encourage since the cache can grow up to 40GB or more) and build like this.

```bash
APPTAINER_CACHEDIR=/home/atuin/username/containers/cache/  # adjust to point to cache directory
apptainer build --fakeroot vllm.sif vllm.def
```

After building you get a vllm.sif file which you can test if it works by running the following test command.

```bash
apptainer test --nv vllm.sif
```

And finally, you can run it like this:


**OpenAI-compatible API server (default):**

```bash
apptainer run --nv \
  --bind /path/to/models:/models \
  vllm.sif \
  --model /models/your-model \
  --host 0.0.0.0 --port 8000
```

**Custom command (override runscript):**

```bash
apptainer exec --nv vllm.sif python -c "import vllm; print(vllm.__version__)"
```



## Lessons learned

For such complicated packages that are highly dependent on multiple big packages and CUDA version, it is almost mandatory to use these packages through their containerized version.

---

> I am open for collaboration. I am interested in solving and talking about problems related to serving AI and the system operations around it. If you have problems related to reliable deployment of AI systems (on cloud, HPC or even on-perm), feel free to write me an email.
