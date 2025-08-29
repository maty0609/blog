---
layout: post
excerpt: "My new home AI server - Part 2 - MOBO, CPU, Memory, Storage etc."
title: My new home AI server - Part 2 - MOBO, CPU, Memory, Storage etc.
categories: [AI, Linux, Homelab]
author: Matyas
published: true
comments: true
---

## AMD vs Intel

I think that has been pretty much decided from the beginning. AMD lately is doing amazing CPUs which are providing not just great performance but also efficiency. If this is going to be server which I will be running home I want to make sure I can run this effeciently as much as I can. Also since I will be focusing on AI/ML I will be testing things like AI inferencing on CPU and AMD based on some benchmarks show EPYC outperforms comparable Intel Xeon Platinum systems in Llama2 and Llama3 model inferencing. Yes I know some Xeon models feature AMX matrix acceleration for AI workloads, which can tip the scales in small/medium-size models but I need something more versatile. 

AMD it is.
## Server vs desktop

Did I already mention I'm building server? When you look on the internet some people build their home servers on desktop platforms - and I'm not surprised when you see what you can run these days on consumer HW: Motherboard supporting 5 PCIe v5, up to 2TB RAM and on top of it supporting WiFi and 10x USB ports? Not a problem...

However, you are still losing on certain features with desktop platform like max cores, memory channels and things like remote management. I'm not going to hit absolute limits of EPYC platform from the beginning but when I will be building new server I want to make sure it will be future proof as much as possible. My long term plan with this home server is it will go to datacenter or it will be running in different place so I need to be able to manage the server remotely.

AMD server platform EPYC will offer support for more cores, more memory channels and more features like IPMI. 

AMD EPYC it is.

## CPU

I don't have financial justification for Dual CPU system like I don't have any financial justification currently for running dual GPU system so let's start with statement that for this built single CPU will be enough. Now what CPU?

Support for PCIe5 and DDR5 is a must to be able to support future requirements. This and my budget will narrow the list to basically two EPYC models. 9124 and 9224.

| Specification | EPYC 9124 | EPYC 9224 |
| --- | --- | --- |
| **Price (Ebay)** | £499 | £780 |
| **Cores/Threads** | 16/32 | 24/48 |
| **Base Clock** | 3.0 GHz | 2.5 GHz |
| **Max Boost Clock** | 3.7 GHz | 3.7 GHz |
| **All-Core Boost** | 3.6 GHz | 3.65 GHz |
| **L1 Cache** | 16 × 32KB (I+D) | 24 × 32KB (I+D) |
| **L2 Cache** | 16 MB | 24 MB |
| **L3 Cache** | 64 MB | 64 MB |
| **Memory Channels** | 12 | 12 |
| **Max Memory** | 6,144 GB | 6,144 GB |
| **Memory Speed** | DDR5-4800 | DDR5-4800 |
| **Memory Bandwidth** | 460.8 GB/s | 460.8 GB/s |
| **PCIe Version** | 5.0 × 128 lanes | 5.0 × 128 lanes |
| **TDP** | 200W | 200W |
| **TDP Up** | 240W | 240W |
| **Socket** | SP5 | SP5 |
| **Manufacturing** | 5nm + 6nm | 5nm + 6nm |


In the nutshell I will be choosing between less cores/higher frequency vs more cores/lower frequency. I have said I'm planning to play with AI inferencing which would indicate to have maybe more cores/lower frequency as AI inferencing requires lots of parallel computing. The jump in price is however over 40% which to me is not really justifiable. I will compromise here and will go for cheaper CPU with enough of performance for my testing and workloads with support of DDR5 and 12 memory channels. Great thing about EPYC 9004 platform is that most motherboards which supports EPYC 9004 (Genoa) usually supports EPYC 9005 (Turin) processors. I don't have currently budget for EPYC 9005 however I will have motherboard, memory and GPU which will allow me to upgrade if I need to.  

AMD EPYC 9124 it is.

## Motherboard

Long story short there is not many server motherboards you can buy let's say on Amazon. Most of your HP, Cisco, Dell servers are using their proprietary motherboards so you have only limited amount of options - mostly Supermicro or ASRock. I have never touched ASRock but based on the feedback it looks kinda punk - customers are complaining about not so great layout and often lacking features like remote management. Supermicro looks tested for many many years and supports remote management. Where do you buy Supermicro? Mostly eBay. Did I ever buy motherboard on eBay? No. Am I scared? Yes. Do I have any other option? Very likely not.

So ignoring the fact I will be ordering motherboard on the slow boat from China what features I'm looking for? I need future proof motherboard - up to 2 GPU slots because I don't think I will need more GPUs for this specific built, DDR5, support for EPYC 9004 and EPYC 9005 for future performance, support PCIe v5 to support any future GPUs, minimum support for 1TB RAM to test CPU inferencing, support remote access, support M.2 for boot/OS storage and U.2 storage for any and SATA at the same time.

After exploring Reddits, home server forums etc. I have found Supermicro H13SSL-N which I think will tick all the boxes. Small caveat it needs to be ver 2.0 which support EPYC 9004 and 9005. Older version support EPYC 9004 only and looks like some people finding them unreliable.

Supermicro H13SSL-N ver 2.0 it is.

## Memory
Supermicro H13SSL-N and EPYC 9004 supports up 3 TB of ECC DDR5/3DS RDIMM at 4800 MT/s so that will help because DDR5 5600MT would probably cost me one of my legs. So when 4800MT are cheaper how much memory I will need? I'm thinking in the range of 200-256GB for now. Do I need them now? Nope. So let's start with 128GB with the intension of expanding it to 256GB in the mid-term. 

One of the reasons I chose EPYC 9004 platform was that it supports up to 12 memory channels. So in the long term I want to be able to fully utilize them. I can go 12x 16GB which will get me to 192GB. I don't feel like 192GB is something I need in "phase 1". 128GB will be probably optimal for this phase so long story short I have decided I will go for 4x 32 GB DDR5s. I will not be able to maximize memory channels but I will have space to expand my RAM in the following months/years up to 384GB RAM with just buying more sticks.

One last thing which is always good to check is the compatibility. I will not be risking of using some no-name vendors but will go for Kingston. They have nice compatibility tool [here](https://www.kingston.com/unitedkingdom/en/memory/search/model/107369/supermicro-h13ssl-n-motherboard?status=active&capacity=32).

4x Kingston 32 GB DDR5 4800MT/s it is.

## Storage
Big big topic - M.2, U.2, SSD, NVMe, HDD, PCIEv4, PCIEv5, RAID1, RAID5 - so many terms that I didn't even where to start.

I have decided to start with what type of storage I will need for day 1.

I will need boot/system storage and than something where I will be able to store "other" data. I need one type of the storage with high performance and the other one probably not so much performance - at least not for now. Everything will have to be mirrored using RAID1.

M.2 NVMe using PCIEv5 is currently the fastest type of storage you can have in your severs. There are alternatives like M.2 NVMe using PCIEv4 which is around 50% cheaper but only about 10-15% slower. In my view PCIEv5 doesn't really worth the extra cost so it will be M.2 NVMe PCIEv4. Capacity wise I think it will be enough to have 2TB for boot/system storage and I don't believe I will need to add any more capacity to this for very ling time.

Now the data storage will be more tricky. The logical option would be U.2 storage but I feel for the start it would be unnecessary overkill. I'm planning to really start hammering the server from in following months so don't need U.2 on day one and at least I can save some money. I have couple of spare 3TB SATA HDDs so I will put them into RAID1 and use them for the data storage. They will be fine for next few months.

2x Samsung 990 PRO NVMe M.2 SSD, 2 TB, PCIe 4.0
2x Seagate 3TB SATA HDD

## Chassis
I want to make sure my chassis will allow me lots of space for upgrades in the future and it is not going to be limitation from performance perspective. At some point I will be moving this server into rack so I want chassis which will be suitable for rack deployment. 1-2 RU are no go for properly sized server with multiple modern GPUs. I was looking at 3RU and it was limiting from CPU cooler perspective (AMD EPYC 9004 coolers are HUGE!) so it will need to be 4RU. I also like the idea of 4RU because that will provide enough of space for the future upgrades and enough of space for proper cooling and tinkering with my new server.

After some googling I have found that Sliger builds solid and beautiful server chassis where you can even change the front panel! Beautiful. Sliger CX4200a looks like the winner and I'm going to buy it in Carmin Red - because red makes it event faster.

It is Sliger CX4200a in RED.

## Miscellaneous 
Chassis fans - 4x Noctua NF-A12x25 PWM
CPU fan - Noctua NH-D12L
PSU - Seasonic PRIME TX-1600

## Final specs - Please welcome my new AI rig

| Type                  | Part Name                                                |
| --------------------- | -------------------------------------------------------- |
| **Motherboard**       | Supermicro H13SSL-N                                      |
| **CPU**               | AMD EPYC 9124                                            |
| **Memory**            | 128GB DDR5 ECC (4 x 32GB)                                |
| **Chassis**           | Sliger CX4200a                                           |
| **Chassis**           | Sliger 4U Replacement Front Panel with HEX pattern - RED |
| **Chassis**           | 4U Rear Exhaust Fan Mounting Bracket 120mm               |
| **Chassis Front Fan** | 3x Noctua NF-A12x25 PWM                                  |
| **Chassis Rear Fan**  | Noctua NF-A12x25 PWM                                     |
| **CPU Fan**           | Noctua NH-D12L                                           |
| **CPU Fan**           | Noctua NT-H2                                             |
| **GPU**               | RTX 3090                                                 |
| **PSU**               | Seasonic PRIME TX-1600                                   |
| **Boot/OS storage**   | 2x Samsung 990 PRO NVMe M.2 SSD, 2 TB, PCIe 4.0          |
| **UPS**               | APC BR1500GI - 1500VA                                    |
| **Storage**           | 2x 3TB SATA HDD                                          |

## Links

* [AMD SP5 Genoa Motherboards from the USA now available on Newegg](https://forum.level1techs.com/t/amd-sp5-genoa-motherboards-from-the-usa-now-available-on-newegg/193980)
* [AI Server is up!](https://www.reddit.com/r/LocalAIServers/comments/1lugjvy/ai_server_is_up/)
* [Compare AMD EPYC 9124 and 9224](https://www.perplexity.ai/search/compare-amd-epyc-9124-and-9224-8K6xqztfSzeIPIb4bZbtpw)
* [EPYC CPU Build: Which CPU to choose?](https://www.reddit.com/r/LocalLLaMA/comments/1lmr1qh/epyc_cpu_build_which_cpu_9354_9534_9654/)
* [Asrock vs Supermicro AMD EPYC Genoa](https://www.reddit.com/r/truenas/comments/1dekxey/asrock_vs_supermicro_amd_epyc_genoa/)
* [Antec Performance 1 FT Review](https://aphnetworks.com/reviews/antec-performance-1-ft)
* [Which 2TB NVMe SSD for a Homelab?](https://www.reddit.com/r/homelab/comments/15xhf9y/which_2tb_nvme_ssd_for_a_homelab/)
* [Time to build more servers, suggestions needed!](https://www.reddit.com/r/LocalAIServers/comments/1k54qtk/time_to_build_more_servers_suggestions_needed/)
* [Building a server for AI, I have a...](https://www.perplexity.ai/search/i-m-building-server-for-ai-i-h-QT3FpQY4RYa5WKcrNadN2g)
* [EPYC 9005 Turin Motherboard Options](https://forum.level1techs.com/t/epyc-9005-turin-motherboard-options/224237/2)
* [Compare AMD EPYC 9124 and 9224](https://www.perplexity.ai/search/compare-amd-epyc-9124-and-9224-8K6xqztfSzeIPIb4bZbtpw)
* [DENSITY CX3170A XL Rackmount Case (3U)](https://www.density.sk/cx3170a-xl-rackmount-case-3u/)
* [YouTube Video: Sliger CX4712 TrueNAS Box Showoff](https://www.youtube.com/watch?v=91dp5l44X8A)
* [My Sliger Rack Mount Cases for Home Office Lab](https://www.reddit.com/r/sliger/comments/12blyri/home_officelab_with_new_sliger_rack_mount_cases/)
* [Sliger Mods](https://esologic.com/sliger-mods/)
* [Kingston Memory Search: 32GB DDR5 for Supermicro H13SSL-N Motherboard](https://www.kingston.com/unitedkingdom/en/memory/search/model/107369/supermicro-h13ssl-n-motherboard?status=active&capacity=32)
* [Which memory for the Supermicro H13SSL with an EPYC 9115?](https://forum.level1techs.com/t/which-memory-for-the-supermicro-h13ssl-with-an-epyc-9115/225180)
* [Phoronix Review: Supermicro H13SSLN EPYC Turin](https://www.phoronix.com/review/supermicro-h13ssln-epyc-turin)
* [Single AMD EPYC 9004 Workstation from Superworkstations](https://superworkstations.com/products/single-amd-epyc-9004-workstation/)
* [ECC Memory Options for Server Premier ECC Hynix A 16GB DDR5](https://ecprof.com/memory/ksm48r40bs8-16ha-cl40-server-premier-ecc-hynix-a-16-gb-ddr5.html)
* [Kingston KSM48R40BD8-32HA KSM 32GB DDR5 4800 ECC REG DIMM](https://ecprof.com/memory/ksm48r40bd8-32ha-ksm-32gb-ddr5-4800-eccreg-dimm.html)
* [AMD EPYC 7302P Supermicro H11SSL-I Version 2](https://forums.servethehome.com/index.php?threads/amd-epyc-7302p-supermicro-h11ssl-i-version-2.37913/page-23)
* [PCIe 50 NVMe Worth It?](https://www.reddit.com/r/LocalLLaMA/comments/165ann4/pcie_50_nvme_worth_it/)
* [Supermicro H13SSL-N and fans oscillating between low and high speed](https://forums.servethehome.com/index.php?threads/supermicro-h13ssl-n-and-fans-oscillating-between-low-and-high-speed.41246/)


