---
layout: post
excerpt: "Building AI Data platforms with WEKA"
title: "Building AI Data platforms with WEKA"
categories: [WEKA, CFD]
author: Matyas
published: true
comments: true
---

&nbsp;
<img src="../../img/2023-10-25-building-ai-data-platforms-weka/logo.png"
     alt="" width="500" align="center" />

In the realm of AI and Machine Learning (AI/ML) as well as image processing, the necessity for high-performance storage cannot be stressed enough. This involves high I/O capabilities with minimal latency, ensuring that operations are swift and seamless.

Traditionally, many setups, whether in cloud environments like AWS, Azure, GCP, or Oracle, or on-premises, have been plagued by a primary challenge: storage that struggles to feed compute adequately. This issue is further intensified when when you are for instance company who is focusing on AI building LLM. These companies make significant investments in GPUs in public cloud, but often, the storage bottlenecks, characterized by prolonged load/write times, prevent them from achieving optimal performance.

To address these challenges, a more streamlined AI data pipeline is crucial. A typical workflow might look something like this: Data Ingestion => Metadata lookups => High I/O operations => Data storage. The ultimate goal? To feed GPUs faster, potentially reducing the number of GPUs required or do more with the same amount of GPUs.

One innovative solution to address exactly these challenges is WEKA. WEKA leverages the NVMe storage in AWS, Azure, GCP and Oracle instances and than aggregates 80-90% of the data of storage to i.e. S3 or Azure Blob. This pool is called WEKA namespace which creates object storage available to endpoints outside of WEKA namespace. It has been benchmarked to achieve an impressive 5M IOPs in AWS with 40X faster model deployment. Pretty impressive!
There are significant cost savings tied to this approach as well. Organizations can sidestep the need to maintain multiple instances, thanks to features like autoscaling and data reduction. Furthermore, the flexibility of such a system allows for hybrid environments, functioning both on-premises and in the cloud or even spanning multiple cloud providers. Whether you're looking at brownfield (existing) or greenfield (new) deployments, the system is designed to support both.

Auto-scaling is made even more efficient through defined groups in the cloud, and the data tiering in S3 offers added advantages. By chopping files into manageable pieces, parallelism is achieved. Meanwhile, smaller files can be merged into larger objects, optimizing storage. Pretty neat!

The support extends to multi-AZ deployments, with the only current limitation being the non-movement of data closer to instances but apparently this is something which is being explored. With robust solutions like AWS's P4d and P5d instances, the core of these operations is fortified. 

In case the cost is your priority you can always spin down the whole WEKA cluster with storing the data to your S3 or Azure Blob. Later if you spin up the cluster again it will re-hydrate the data from S3 and you are good to go. 
In conclusion, as AI/ML operations grow in complexity and demand, it's vital for organizations to evaluate their storage solutions critically. Optimal performance isn't just about computational power but ensuring that storage infrastructures can keep pace.
