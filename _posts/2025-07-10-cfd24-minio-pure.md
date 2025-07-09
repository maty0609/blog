---
layout: post
excerpt: "AI storage"
title: AI storage - MinIO AIStor and Pure Storage FlashBlade //EXA
categories: [MinIO, Pure Storage, Cloud Field Day]
author: Matyas
published: true
comments: true
---

In this article I will not be talking about your typical enterprise. I will be talking about scale of OpenAI, Meta, Tesla and other large scale AI companies. I was planning to release this right after MinIO presentation but I didn't because things has changed a week after the presentation with Pure Storage announcing FlashBlade //EXA which has removed some of the limitations. So I thought I will rewrite this to talk little bit more about what challenges MinIO and FlashBlade //EXA trying to solve and zoom in little bit into each product. I wasn't able to test performance any of those solutions so even though performance is very important when it comes to AI storage I will be focusing purely on features and architecture and when it comes to performance I will work with some assumptions. I'm totally aware there are more products than MinIO and FlashBlade out there like WEKA but in this article I will be focusing only on those two.

### AI and storage - new challenges

AI workloads are defined by their huge appetite for data. The sheer volume is staggering, with training datasets for large language models (LLMs) and computer vision systems routinely scaling from terabytes to petabytes and, increasingly, exabytes.

On top of it different AI frameworks and stages of the data pipeline generate vastly different I/O patterns. For instance, frameworks like PyTorch are known to operate on a high number of tiny files, which places immense stress on a storage system's metadata performance and its ability to handle high input/output operations per second (IOPS). Conversely, other frameworks like TensorFlow or stages like checkpointing may involve a smaller number of very large files, demanding high sequential throughput.   

AI storage platform (the word platform is important here because it will hopefully make little bit more sense why do we need storage platforms) must excel at both small-file IOPS and large-file throughput without requiring manual re-tuning or reconfiguration between workloads.

Modern GPUs provide massively parallel processing power, but their efficiency is entirely dependent on a consistent, high-speed stream of data from the storage subsystem. When storage cannot keep pace, GPUs are left idle, waiting for data. This phenomenon, known as "GPU starvation," represents a significant waste of expensive computational resources and can dramatically extend model training times, slowing the pace of innovation. This is something I have mentioned in my post about WEKA couple years ago so nothing has changed here. The quite opposite....   

Preventing GPU starvation is a primary design goal for AI storage. It requires a solution that delivers not only massive aggregate bandwidth but also extremely low latency. Any delay in data access, whether for feeding training pipelines or performing real-time inference, can become a critical bottleneck that slows the entire operation

### The AI Data Pipeline

The AI data pipeline describes the journey of data as it is processed through the various stages of the AI lifecycle. Each stage imposes distinct I/O demands on the underlying storage system, making the pipeline a critical framework for evaluating storage architectures.   

- Ingest: This initial phase involves collecting raw data from various resources. The data is often unstructured and can amount to petabytes. The storage requirement here is for a highly scalable, cost-effective platform that can handle high-throughput, parallel writes. Object storage is the ideal protocol for this stage due to its scalability and ability to handle unstructured data. Think of your typical Dell or Pure Storage object storage.  

- Preparation: Raw data is rarely usable for training. In this phase, it is cleaned, transformed, labelled, and formatted. This process involves intensive, mixed I/O patterns with frequent reads and writes. The storage must provide high-performance access to what is typically a subset of the raw data lake. Both high-performance file and object storage can be used here. Now this is becoming more interesting because you will need to start looking at something more performing so I would be probably looking here at Pure Storage FlashBlade which supports high IOPS both for file and object. FlashBlade UFFO (more on this later) is really perfect fit for this.

- Training & Checkpointing: This is the most computationally and I/O-intensive phase. Massive, curated datasets are fed to GPU clusters to train the model. This requires a storage system capable of delivering extreme, parallel read throughput with very low latency to keep GPUs saturated. High-performance parallel file systems are the traditional choice for this stage. Concurrently, the training process generates periodic "checkpoints"—snapshots of the model's state—which involve large, sequential writes to storage to ensure resilience against failures. Now this is becoming more interesting because you will need to start looking at something more performing so I would be probably looking here at Pure Storage FlashBlade //EXA in combination with Portworx CSI to ensure you have something which support high IOPS and at the same time integrated with your Kubernetes environment. However, this can be also perfect place for MinIO AIStor which offers high IOPS but at the same time lots of good features on top of it. There are obviously different approaches where it feels Pure Storage FlashBlade //EXA focuses on raw performance MinIO is trying to be a bit more smarter and introduce more features and optimizations on top of it i.e. leveraging ARM SIMD capabilities (more on this later)

- Inference & RAG: Once a model is trained, it is deployed for inference, where it makes predictions on new data. For real-time applications, this requires the lowest possible latency access to the model files. In RAG workflows, the model's predictions are augmented with information retrieved from an external knowledge base, often stored in a vector database. This adds another low-latency read requirement to the pipeline.

### MinIO AIStor
- MinIO has established itself championing a software-defined, hardware-agnostic, and object-native storage. There is history of storage companies who used their code - legaly or not so legally but that's for another time....

- Highly efficient and remarkably lightweight binary of less than 100 MB. This allows MinIO to be deployed in a vast range of environments, from edge devices to exabyte-scale data lakes.

- It can run on commodity infrastructure, directly challenging the economics and architectural rigidity of traditional storage appliances. But I would say it all depends.... Yes running software on commodity hardware can challenge status quo of traditional architecture of running compute and storage in disaggregate or converged architecture but that in my experience works to certain extent. Hyperconverged vendors often uses the same logic saying hyperconverged is "the way hyperscalers runs their datacenters" so it must be the same for the typical enterprise however they forgot to say hyperscaler's commodity hardware is next level from commodity hardware - they build their own racks, chassis, software etc which is not something the typical enterprise would do. Typical enterprise will buy cheap Dell pizza box servers which still means that if you need more storage for your backups or hyperconverged cluster you are buying servers with CPUs, motherboard, power supplies, support and that can cost you in the end more money than expanding your traditional storage. However, try to start deploy cluster with your traditional storage with let's say initial requirement 1TB. No storage vendor will probably be able to help you with such small requirement so scaling with software solutions like MinIO could be the way. 

- MinIO generally recommends keeping compute (GPU clusters) and storage (AIStor clusters) separate. While local GPUs might seem appealing, it limits independent scaling of compute and data. MinIO argues that the network, optimized with technologies like GPU Direct, is the effective bridge, preventing compute nodes from sitting idle as data grows independently, a lesson learned from Hadoop's challenges

- MinIO has optimized its software with Neon (ARM SIMD instructions). This enables building low-power, high-performance servers that are essentially a JBOD (Just a Bunch Of Disks) connected directly to a smart NIC. MinIO runs natively on ARM-based DPUs like NVIDIA BlueField-3, leveraging ARM SIMD capabilities to offload and accelerate storage tasks in modern AI architectures, thus providing scalable, efficient, and future-ready object storage tailored for AI and ML applications

- MinIO has been working closely with NVIDIA on GPU Direct Object, an interface that bypasses the CPU and host memory to directly read and write data from storage into GPU memory

- MinIO integrates with Kubernetes via Custom Resource Definitions (CRDs) for provisioning object storage tenants, buckets, and applying policies. Natively which is difference from other vendors.

- MinIO AI Hub allows enterprises to store and manage their fine-tuned models developed from open-source base models using their private data sets. This addresses the need for specialized use cases without exposing proprietary information. AI Hub is fully compatible with Hugging Face, which is a de-facto standard in the AI world for models and datasets

### Pure Storage FlashBlade //EXA

- Pure Storage approaches the AI storage challenge from a fundamentally different perspective than its software-defined counterparts. The company's philosophy is centered on delivering a vertically integrated, appliance-based solution that co-designs hardware and software to provide extreme performance with unparalleled simplicity, reliability, and a predictable, turnkey operational experience. Its flagship scale-out platform, FlashBlade, culminating in the new FlashBlade//EXA, is purpose-built to serve as a unified data hub for the entire AI pipeline. FlashBlade //EXA architecture allows independent scaling of data and metadata nodes. FlashBlade//EXA data nodes are off-the-shelf (OTS) servers optimized for seamless integration with FlashBlade//EXA metadata nodes which is big shift from previous FlashBlade architecture. It almost raising question why Pure would still call this FlashBlade?

- Vertically Integrated Design: Pure Storage designs and builds its own hardware modules and Purity software runs on them. This end-to-end control allows for deep optimization that is not possible when layering software on top of commodity components. The result is an appliance model that is easy to deploy, manage, and support, as there is a single vendor responsible for the entire stack. This is obviously relevant in case of FlashBlade //EXA only for meta nodes. Data nodes are off the shelf running “thin” Linux-based OS and kernel with volume management and RDMA target services that are customized to work with metadata residing on the FlashBlade//EXA array. With this approach FlashBlade //EXA removes any limitations for data which has been in place in case of FlashBlade //S or //X.

- DirectFlash Technology: A key enabler of this integration is Pure's DirectFlash technology. Instead of using standard off-the-shelf SSDs, which have their own internal firmware (Flash Translation Layer), Pure builds its own DirectFlash Modules (DFMs). This allows its Purity operating system to communicate directly with the raw NAND flash media.  

- Unified Fast File and Object (UFFO): The FlashBlade platform is architected as a UFFO storage system, designed to consolidate data silos by serving both file (NFS) and object (S3) protocols from a single, high-performance platform. This multi-protocol capability enables diverse workload consolidation but may introduce performance trade-offs compared to single-protocol optimized systems. 

### Summary

In both cases we see a new generation of storage built around GPUs and not linked to CPUs like we used to. The architecture is changing with MinIO pioneering the architecture and flash leader Pure Storage following trends. So what's the summary?

MinIO excels in implementations through S3 compatibility and horizontal scaling capabilities. The architecture supports natively diverse data frameworks including Apache Spark, Presto, and Hadoop ecosystem components. Cost-effective scaling enables organizations to build petabyte-scale data lakes on commodity infrastructure.

FlashBlade //EXA provides high-performance data lake capabilities with multi-protocol access enabling diverse analytical tools and frameworks. The unified file and object approach simplifies data lake architecture but may require higher infrastructure investment.

MinIO demonstrates flexibility for cloud-native and edge deployments through its lightweight architecture and multi-platform support. The solution supports deployment across public cloud, private cloud, and edge environments with consistent S3 API access. Recent NVIDIA and ARM processor optimizations enhance edge computing capabilities with improved power efficiency.

FlashBlade //EXA primarily targets on-premises and data center deployments, with limited cloud-native flexibility due to hardware appliance requirements. The solution excels in traditional enterprise environments but faces constraints in distributed cloud architectures.

MinIO requires moderate infrastructure expertise for optimal deployment and management, particularly in large-scale distributed configurations. Organizations benefit from extensive documentation, community support, and professional services for complex deployments.

FlashBlade //EXA simplifies management through integrated Purity operating system and web-based graphical interface. The appliance approach eliminates hardware compatibility concerns and provides predictable management experiences. FlashBlade as part of larger Pure Storage platform simplify the operations.

My summary would be that if you are company who already owns and manage large cluster of servers and want to decentralize your AI/ML infrastructure with i.e. finetuning your models on the edge or in the cloud and looking for laser focused AI storage solution with high performance MinIO will be good fit for you. If you are large enterprise which is building AI Factory and will be building or finetuning very large language models and at the same time looking for something which is more universal FlashBlade //EXA will be the more suitable solution for you.


