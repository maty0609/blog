---
layout: post
excerpt: "My new home AI server - Intro, requirements and selecting the right GPU"
title: My new home AI server - Intro, requirements and selecting the right GPU
categories: [AI, Linux, Homelab]
author: Matyas
published: true
comments: true
---

So far there were 2 big transitions in the networking space during my lifetime - internet and cloud. I was lucky enough to be part of internet transition end of 90s and beginning in 00s building the community internet provider Bubakov.net and oh how much I have learnt when building it! We used to take PCs, install Linux on them and run them as servers, routers and switches. We were learning how to do routing, deploy BGP or to deploy IPv6. It was absolute blast.

Over a decade later I had lots of fun in Google and Natilik in mid 2010s building cloud infrastructure and my cloud skills. I was able to get lots of hands on as engineer, architect but also last few years as cloud practice leader and I do believe we are now in the next big transition in networking space caused by AI.

I'm simple person. If I really want to understand something I have to build it, break it, touch it and so I feel like the only way to really understand how to build AI infrastructure I will have to start learning how to build it with my own hands. So I have decided that I will be the first time since 2005 build the server and I can't be more excited about it!

This is my private project so my budget will be limited. This is a good thing. I like limitations and constraints because they really push you to think, evaluate and assess things. I need to find the right balance between the performance I need and the final cost.

I wanted to write this post mainly to give me a structure for assessing different GPUs and who knows maybe it will help to other people who are going through the same process.

## My Home Lab mission statement

Let's start to define what do I want to get out of this?

1. I want to have fun!
2. Start small - I don't want to invest massive amount of money at the start and found out this is useless - this needs to be approached as PoC.
3. I want to learn a lot about new HW, SW, products etc - end goal is to be able to build AI Server and eventually sell it if needed.
4. Improve my knowledge about AI/ML in general - I want to learn the whole stack from the infrastructure all the way to the top with writing CUDA kernels
5. I'm NOT planning to replace Gemini or Perplexity by this. I don't have expectations this will allow me to run such large and capable models
6. Make sure there will be at least some ROI at the beginning I will migrate some of my cloud services to this - mail server, web server, TeslaFi, HomeAssistant, NextCloud and VPN

## My Home Lab AI/ML Requirements

How do I want to use this server for my AI/ML experiments?

1. Inferencing - running LLM locally (80% of the time)
2. Finetuning and training - Train video models, finetune LLM models (20% of the time)
3. Learn more about Nvidia - CUDA, run NIMs etc.
4. GenAI - Generate pictures and videos

## GPU List

As this is going to be primarily AI home server I felt it will be the best to start with selecting the right GPU. Based on that I will decide other components. I wanted to make sure I will choose the best GPU based on price/performance ratio which means I don't need the best performing GPU but at the same time don't want to buy GPU only based on the lowest cost. It needs to sit somewhere in the middle. 

Before I have started to do decide which GPU to select I have built table with the most popular GPU for home AI servers. I didn't really look at server GPUs because that's not where my budget is. I felt that RTX series will be much more suitable.

| Model | Price | Graphics Co-Processor | CUDA Cores | Graphics RAM Size | Memory Bandwidth | NVLink Support | CUDA Compute Capability | Tokens/sec (Llama 8B) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| RTX PRO 4000 Blackwell | Not Available | NVIDIA GB203 (Blackwell-Pro) | 8,960 (280 Tensor / 70 RT) | 24GB GDDR7 - 192-bit | 672GB/s | No | 12.0 (CUDA 12.8+) | 59 t/s |
| RTX 4500 Ada (Pro) | £3,084.52 | NVIDIA AD103 (Ada) | 7,680 (240 Tensor / 60 RT) | 24GB GDDR6 - 192-bit | 432GB/s | No | 8.9 (CUDA 11.8+) | 60 t/s |
| RTX 5080 | £1,345.00 | NVIDIA GB203 (Blackwell) | 10,752 (336 Tensor / 84 RT) | 16GB GDDR7 - 256-bit | 896GB/s | No | 12.0 (CUDA 12.8+) | 132 t/s |
| RTX 5090 | £2,200.00 | NVIDIA GB202 (Blackwell) | 21,760 (680 Tensor / 170 RT) | 32GB GDDR7 - 512-bit | 1,792GB/s | No | 12.0 (CUDA 12.8+) | 150 t/s |
| RTX 3090 | £1,250.00 | NVIDIA GA102 (Ampere) | 10,496 (328 Tensor / 82 RT) | 24GB GDDR6X - 384-bit | 936GB/s | Yes | 8.6 (CUDA 11.1+) | 112 t/s |
| RTX 4090 | £2,800.00 | NVIDIA AD102 (Ada Lovelace) | 16,384 (512 Tensor / 128 RT) | 24GB GDDR6X - 384-bit | 1,008GB/s | No | 8.9 (CUDA 11.8+) | 128 t/s |
| DGX Spark | £3,500.00 | GB10 Grace Blackwell Superchip | Blackwell Gen / 5th Gen Tensor | 128GB LPDDR5x Unified (CPU+GPU) | 273GB/s | No | CUDA 12.8+ | ~7.8 t/s (est. Llama 70B Q8) |
| Tesla P40 | £1,856.16 | NVIDIA GP102 (Pascal, 16nm) | 3,840 | 24GB GDDR5 - 384-bit | 346GB/s | Yes | 6.1 (CUDA 8.0+) | ~30-40 t/s |
| L40 | £7,119.98 | NVIDIA AD102 (Ada Lovelace) | 18,176 (568 Tensor/142 RT) | 48GB GDDR6 ECC - 384-bit | 864GB/s | No | 8.9 (CUDA 11.8+) | ~90–105 t/s |

Based on these details, I have shortlisted my selection to two GPUs: RTX 5090 (representing higher price but also higher performance) and RTX 3090 (representing middle priced GPU with above the average performance). Now I will assess each GPU against "required deliverables" when it comes to AI/ML.

## Req #1 - Inferencing - running LLM locally
What do you need to consider when selecting GPU for finetuning?

### Memory Capacity (VRAM)
Memory can be the only limitations when it comes to how large models you can run. Ideally you want to run your model from GPU VRAM. If the LLM doesn't fit in VRAM you will see a loss of speed due to RAM/VRAM transfers or CPU fallback. Or in the worst case failure...

So would be huge difference for me running experiments with RTX 3090 of 24GB VRAM or RTX 5090 of 32GB VRAM? Probably not. 

🏆 RTX 3090 is the winner. 🏆 

### Compute Performance
For inferencing, raw computational power (measured in TFLOPS or TOPS) is still relevant, but typically less critical for training and I would say that even more non-critical for me experimenting with AI/ML. It will be fine to handle single user requests or run other data models maybe little bit slower with RTX 3090 than with RTX 5090 and save some money.

🏆 RTX 3090 is the winner. 🏆 

### Memory Bandwidth
High memory bandwidth ensures smoother performance, particularly with large models or datasets. This enables faster data movement, preventing bottlenecks. Would I love to run 200B models? Absolutely. Does it make sense financially? Definitely not.

🏆 RTX 3090 is the winner. 🏆 

## Req #2 - Finetuning - finetune LLM models
What do you need to consider when selecting GPU for finetuning?

### LOTS of memory to get everything you need into the memory

When it comes to finetuning you have to squeeze into the memory lots of things - Shared Backbone Weights, Activations, Optimizer States, Gradients, Temp Buffers, Framework Overhead and Multi-GPU Overhead. When it comes to how much memory you will need the good rule of thumb is 16GB per 1B rule. With using technique like LoRa this ratio can be even more improved.

If I would be buying single GPU day 1 I will be able to finetune larger models with RTX 5090 with 32GB VRAM than with RTX 3090 with only 24GB VRAM. However, if I will use LoRa I still be able to finetune smaller 7B or even 8B (small batch size like 2 or 3) with RTX 3090. I think for my first little experiments this is sufficient. If this will not be going to be enough I can buy another RTX 3090 or I can always upgrade to 5090....

🏆 RTX 3090 is the winner. 🏆 

### GPU Compute Capability (FLOPS and memory bandwidth)

Speed of your hardware doesn't matter nearly as much as the techniques you apply. VRAM should be the focus of your decision.

Basically if you don't have enough of memory there is no way you will get around finetuning your models. If you will have less performant GPUs the only thing will happened is that your finetuning will take a bit a longer. You can say that running job longer means less efficient and therefore more power consumption and you are right however this would be case of datacentre scale. For my little experiments this is not an issue. Solved.

🏆 RTX 3090 is the winner. 🏆 

### Interconnect Bandwidth - NVlink vs PCIe Gen4/5

This is a tricky one. You will see lots of people giving their benchmarks with various results.

Like I have stated mostly I will be using this for inferencing so what impact NVlink vs PCIe Gen4/5 will have? NVlink will have minor improvement for inferencing so no real advantage over PCIe however there seems solid proof that NVlink has huge improvements on finetuning performance over PCIe. Would I see more performance with RTX 5090 with 32GB VRAM over 2x RTX 3090 with total 48GB VRAM? Probably. Would I be able to actually do any more finetuning with single RTX 5090 than with single RTX 3090. Yes I would - slightly larger models. Since I will be able to test any finetuning concepts with smaller models I think it is save to say that RTX 3090 will be sufficient.

🏆 RTX 3090 is the winner. 🏆 

### FP16/BF16 mixed precision, INT8
Not really relevant for my use case at this point. I'm planning to use LoRa and LoRA which uses FP16/BF16 fixed base model weights.

🏆 RTX 3090 is the winner. 🏆 

### Number of GPUs
When a single GPU's resources are insufficient, the ability to use multiple GPUs in parallel (using DDP, FSDP, or ZeRO optimizations) is essential. The total available VRAM and aggregate compute power scale with the number of GPUs.

I think this will be more relevant when selecting the right motherboard, chassis and memories. I can run in parallel multiple RTX 3090s or 5090s so it doesn't really have any weight when selecting the right GPU.

## Req #3 - Learn more about Nvidia - CUDA, run NIMs etc.

RTX 3090 with CUDA compute capability 8.6 (Ampere, like RTX 30-series) do not support many new features introduced with CUDA 12.0, which are available only on newer hardware generations like RTX 5090 which runs on Blackwell. 

Key differences include:

- No support for FP8 Tensor Cores and 32x Ultra xMMA operations: These are new features in Hopper/Ada Lovelace, exposed only to compute 9.0+ devices, improving AI, deep learning, and matrix operations.

- No revamped CUDA Dynamic Parallelism APIs: CUDA 12.0 brings major enhancements to dynamic parallelism APIs, with significant speedups and optimizations that are not available or are limited on Ampere GPUs.

- No Hopper/Ada-exclusive memory features: Features like TMA (Tensor Memory Accelerator) operations, advanced L2 cache management (i.e., programmatic L2 cache to SM multicast), extended memory barriers, and asynchronous transaction barriers are only supported on Hopper and up.

- No new genomics/DPX instructions: These are specialized instructions for genomics and dynamic programming, offered only on Hopper architecture (compute 9.0+).

- Limited CUDA Graphs API features: CUDA 12.0 exposes new ways to launch graphs from device code (GPU-side graph launches), giving much more flexibility for kernel scheduling—this is unavailable or restricted on 8.6 GPUs.

- No C++20 or latest compiler features: CUDA 12.0 allows targeting C++20 and newer GCC versions, but only on compatible new hardware/OS platforms.

- Lack of support for other new APIs and developer features: Several new memory management, stream prioritization, and context-unique ID APIs are only available on CUDA 12.0 and thus not accessible on hardware stuck at 8.6

Right now at this point of time I can't say these are real limitations to me. Maybe they will be in the future and at least I'm aware of them but let's cross that bridge when I will get there. For now I'm fine with these limitations and limited features I will be able to test with RTX 3090.

🏆 RTX 3090 is the winner. 🏆 

## Req #4 - GenAI - Generate pictures and videos

I will want to run typical text to picture or video models i.e. Flux.1-dev, Hunyuan Video and Stable Diffusion 3.5. The good news is that you can run these models on either RTX 3090 or RTX 5090. The difference will be in how quickly they will be able to generate pictures. Based on most of the tests on the internet it seems like there is approximately 170% increase in performance between RTX 3090 and RTX 5090 which means 2.7 times faster. Even though the price is approx. 2 times higher I don't feel like this would justify the extra funding.

🏆 RTX 3090 is the winner. 🏆 

## Choosing the right GPU

🏆  The winner is: RTX 3090! 🏆 


## What's next?
I will be looking in following weeks on things like server chassis, motherboards, CPUs, disks, memories etc. I will write follow up posts on to cover my thinking process on other components.

## Links
Here is the list of links rebuilt in markdown format:

* [NVIDIA Cuda Toolkit 12.0 Released for General Availability](https://developer.nvidia.com/blog/cuda-toolkit-12-0-released-for-general-availability/)
* [What Limitations I Would Encount](https://www.perplexity.ai/search/what-limitations-i-would-encou-cThTzBJFQRGUO4gqe9rkiA#2)
* [PSA: NVLink boosts training performance by a lot](https://www.reddit.com/r/LocalLLaMA/comments/1epnppd/psa_nvlink_boosts_training_performance_by_a_lot/)
* [LLM Fine-Tuning GPU Guide](https://www.runpod.io/blog/llm-fine-tuning-gpu-guide)
* [RTX 5090 32GB VRAM Full Fine-Tuning: What Can I Expect?](https://www.reddit.com/r/LocalLLaMA/comments/1m5ro7s/rtx_5090_32gb_vram_full_finetuning_what_can_i/)
* [VRAM Calculator](https://apxml.com/tools/vram-calculator)
* [ArXiv - Efficient Neural Network Training on GPU with NVIDIA's Tesla V100](https://arxiv.org/html/2406.02290v2)
* [My HomeLab Build: 4x RTX 3090 Powerhouse](https://www.reddit.com/r/LocalLLaMA/comments/1ha3hwu/my_homelab_build_4x_rtx_3090_powerhouse/)
* [Benchmark Results: Dual GPU Boosts Speed Despite All Common Caches](https://www.reddit.com/r/LocalLLaMA/comments/1jobe0u/benchmark_dualgpu_boosts_speed_despire_all_common/)
* [Benchmark Results: PCIe 40x16 NVLink 3090 vs 4090](https://www.reddit.com/r/LocalLLaMA/comments/1jf0wvz/benchmark_results_pcie40_1x4x8x16xnvlink_30904090/)
* [Asking for NVIDIA Blackwell RTX 5090 or 5080 for $30B](https://www.reddit.com/r/LocalLLaMA/comments/1if2ve8/asking_for_nvidia_blackwell_rtx_50905080_for_30b/)
* [Comment on Home Server Build with NVIDIA GeForce RTX 3090](https://www.reddit.com/r/HomeServer/comments/1llibw8/comment/n031etp/?context=3&share_id=pK1uUq4earBpmvT5nS9rH&utm_content=1&utm_medium=ios_app&utm_name=ioscss&utm_source=share&utm_term=1)
* [IA Server Finally Done](https://www.reddit.com/r/LocalAIServers/comments/1llj00s/ia_server_finally_done/)
* [Is a 4x or 8x 3090 Build Viable for Both Training and Inference?](https://www.reddit.com/r/LocalLLaMA/comments/1c6y82z/is_a_4x_or_8x_3090_build_viable_for_both_training/)
* [VLLM Performance Benchmarks: 4x RTX 3090](https://himeshp.blogspot.com/2025/03/vllm-performance-benchmarks-4x-rtx-3090.html)
* [Quad 4090, 48GB, 768GB DDR5 in Jonsbo N5 Case](https://www.reddit.com/r/LocalLLaMA/comments/1m9uwxg/quad_4090_48gb_768gb_ddr5_in_jonsbo_n5_case/)
* [NVLink Improves Dual RTX 3090 Inference Performance](https://old.reddit.com/r/LocalLLaMA/comments/1j8i9rc/nvlink_improves_dual_rtx_3090_inference/)
* [Myth About NVLink](https://www.reddit.com/r/LocalLLaMA/comments/1br6yol/myth_about_nvlink/)
* [RTX 5090 vs RTX 3090: Round 2 - Flux 1dev Hunyuan Video](https://www.reddit.com/r/StableDiffusion/comments/1iv6mid/rtx_5090_vs_3090_round_2_flux1dev_hunyuanvideo/)


