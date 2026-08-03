TLDR; The parallelism you chose (the way you load your matrices on GPUs) can make the difference between utilization of 50% and 90% when serving LLM models.

If you have ever used vllm or sglang, you probably came across terms like *tensor parallel* or *pipeline parallel* inference. The reason you decide for one over the other is the number of GPUs and the amount of available VRAM on each GPU.

The rule of thumb:

1. One GPU and the model fits on it? Just use the GPU.
2. One node, multiple GPUs:
 - Model fits on 1 GPU and you want to maximize throughput → data parallelism
 - Model does NOT fit on a single GPU → tensor parallelism
3. Model too large for a single node → combine tensor parallelism with pipeline parallelism.

The 3 types, in one line each:

→ Pipeline parallelism splits the model by its layers. 30 layers, 3 GPUs? Layers 0-9 on GPU 1, 10-19 on GPU 2, and 20-29 on GPU 3. The catch: one GPU waits for the previous one to finish and pass the data. Those small pauses are known as pipeline bubbles.

→ Tensor parallelism splits the matrices themselves. Each GPU holds one part of each layer.

→ Data parallelism replicates the entire model on every GPU. This one is a bit different than the other two: with pipeline and tensor parallelism you cannot get ANY work done otherwise, because the model does not fit. With data parallelism you use more hardware purely to increase throughput. A load balancer then decides who handles the next incoming request (vllm ships with an internal one).

My own experience with this:

I needed to generate 4,000 data entries with Llama-3.2-3B-Instruct on a single node with 4xA40 NVIDIA GPUs, on an HPC environment provided by NHR@FAU. I started with pipeline parallelism and the utilization was way off what I was expecting.

I switched to tensor parallelism and GPU utilization went up to 90% and stayed there.

And here is the mistake I only realized while writing the blog post: I should have used DATA parallelism, not tensor. The model fit perfectly on one GPU, and tensor parallelism adds GPU communication overhead. I could simply have ~4X'd the throughput.

Lesson learned: how you set your model distribution is extremely important in fully utilizing your resources. You should usually aim for +90% utilization of your GPUs. If 90%+ is not achieved, something is wrong. Of course, some use cases make that impossible, but aim for the stars anyway.

This post was less than half of what I wrote in the original post in my blog. If you made it this far, you might find more useful information reading the full blog post [link] as well. 

#LLM #vLLM #GPU #HPC #MachineLearning #Inference
