---
layout: post
excerpt: "Quick summary on Pure//Accelerate 2023"
title: Quick summary on Pure//Accelerate 2023
categories: [PureStorage, Conference]
author: Matyas
published: true
comments: true
---

&nbsp;
<img src="../../img/2023-06-16-quick-summary-on-pure-accelerate-2023/pure.jpeg"
     alt="Pure//Accelerate 2023" width="500" align="center" />

Before I will leave Pure Accelerate I wanted to quickly put together few thoughts on what I have seen this week in Las Vegas. This is quick dump which I have done in an hour so apologies for any typos. You could say that the only takeaway from Pure Accelerate should be that flash is displacing HDD which after many years of repeating the same message got little bit….flat but let’s not be cynical because if you look little bit closer there are some exciting innovations which are happening across PureStorage portfolio which will shape their portfolio for many following years.

I have listed ones which I think are the most impactful I have seen this week in Las Vegas:

### 75 TB DFMs
75 TBs on a single DFM and 150TBs on single DFMs will be announce this year - yeap! This will enable PureStorage to increase the density of the current FAs and squeeze the price per GB even more. Performance increase is impressive - 40% increase in performance for //X series and 60% increase in performance for //E series. Series //X will now have up to 1.5 PB (!!!!) raw capacity in 3RU physical factor. Controllers have been upgraded so you get Sapphire Rapids, DDR5 and support for PCIe5. With higher density it will obviously won’t make sense to maintain //X10s so say goodbye to good old //X10s. Higher density goes hand in hand with better stats on Ws/TBs - //X series improvement approx. 97% and //C series approx. 48%. Will this be enough especially when this year NVMe EDSFF should be able to close the gap from a capacity standpoint? I guess we will see!

### Block storage in cloud
It feels like Cloud Block Store (CBS) maybe undergoes little bit of renaissance. Maybe it was the timing which wasn’t right for the product when it was introduced around 2018 or 2019 (can’t remember the year right now) but it looks like there are some new interesting use cases probably due to maturity of the market and clients. SQL databases replication between cloud and on-prem, data reduction and therefore improving cloud spent or to simplify and improve your block store in Azure and AWS which decrease your cloud spent and improves the operation side of your compute in public cloud. Probably the time is right now?

### Kubernetes Object Storage
It is inventible that object storage will enter into Kubernetes world this or next year. You want to move your cloud native databases between cloud providers or your prem environment often using S3 storage as target but right now that’s difficult if not impossible as each cloud provider uses different standards for objects. It feels like Portworx quietly entering object storage arena with Portworx Object Service challenging vendors like MinIO who are defacto leaders in cloud native object storage. Portworx announced Portworx Object Service which feels like a stop gap before COSI (Container Object Storage Interface) will be finalised. Very excited where this is going.

### The way we see and store data is changing
I’m not AI expert so I can’t tell how much AI will change things in our daily life what I’m pretty sure AI is changing is the cloud and datacenter infrastructure. The amount of data which are needed to build LLMs or use generative AIs requires huge amount of compute power and fast and large storage for the data. It will be interesting how these needs will be reflected into the products like Portworx where most of the infrastructure for AI run. Silos between cold, warm or any other types of storage are disappearing - if you are training model your all data has to be available immediately and in very high speed. PureStorage is perfectly positioned however Vast is challenging them mainly in AI/ML space with different architecture for instance garbage collection etc. Time will tell….

### Cloud operating model
Last but not least is the cloud operating model. We in Natilik are always big believers in cloud operating model. We have onboarded HashiCorp who are the pioneers in cloud operating model for many years and helped lots of our clients on this journey. PureStorage always felt little bit behind - not so much from commercial side where Evergreen pay as you go model exists for a while or Portworx acquisition but mainly from operational side. PureStorage was working really hard last year on Fusion which I would call their orchestration tool. Imagine that your developers will need to provision new volume - you can now provide your developers set of Terraform modules which they can integrate into their current Terraform modules and deploy their application without asking “storage guys” to deploy new volume(s). It should also be able to rebalance the workloads dynamically. All this will run as part of Pure1. In the future I hope we will see the support for Portworx.

Overall great event. It has been great to meet people I haven't seen since my last Pure Accelerate 2019 in Austin before the world went mental. Thank you PureStorage for having me and being so opened talk to me about your new products, ideas and innovations. Looking forward to the next one!
