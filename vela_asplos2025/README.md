## Overview
 
This repository contains the publically released artifacts for the paper titled "[Vela: A Virtualized LLM Training System with GPU Direct RoCE](https://dl.acm.org/doi/abs/10.1145/3676641.3716280)" published at [ASPLOS'25](https://www.asplos-conference.org/asplos2025/program.html). 

```
Following is the list of the released artifcations:

- Details for provisioning GPU Direct RoCE capable VMs on a Linux-KVM-QEMU-based based hypervisor optimized of LLM Training workloads (see "vela_asplos2025/vmsetup").

- Pytorch-based benchmarks for the commonly used collective communication operations employed in LLM training workloads. The key communication collective benchmarks are `allreduce-loop.py`, `allgather-loop.py`, and `reduce-scatter.py` (see "vela_asplos2025/pycommbench").

- Code to replicate the potential RAS issue due to passthrough device fuzzing (see "vela_asplos2025/ras").
```


```
Bibtex: 

@inproceedings{10.1145/3676641.3716280,
author = {Mohan, Apoorve and Walkup, Robert and Karacali, Bengi and Chen, Ming-hung and Kayi, Abdullah and Schour, Liran and Salaria, Shweta and Wen, Sophia and Chung, I-hsin and Alim, Abdul and Evangelinos, Constantinos and Luo, Lixiang and Dombrowa, Marc and Schares, Laurent and Sydney, Ali and Maniotis, Pavlos and Koteshwara, Sandhya and Tang, Brent and Belog, Joel and Odaira, Rei and Tarasov, Vasily and Gampel, Eran and Thorstensen, Drew and Gershon, Talia and Seelam, Seetharami},
title = {Vela: A Virtualized LLM Training System with GPU Direct RoCE},
year = {2025},
isbn = {9798400710797},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
url = {https://doi.org/10.1145/3676641.3716280},
doi = {10.1145/3676641.3716280},
abstract = {Vela is a cloud-native system designed for LLM training workloads built using off-the-shelf hardware, Linux KVM-based virtualization, and a virtualized RDMA over Converged Ethernet (RoCE) network. Vela virtual machines (VMs) support peer-to-peer DMA between the GPUs and SRIOV-based network interface. In this paper, we share Vela's key architectural aspects with details from an NVIDIA A100 GPU-based deployment in one of the IBM Cloud data centers. Throughout the paper, we share insights and experiences from designing, building, and operating the system over a ~2.5 year timeframe to highlight the capabilities of readily available software and hardware technologies and the improvement opportunities for future AI systems, thereby making AI infrastructure more accessible to a broader community. As we evaluated the system for performance at ~1500 GPU scale, we achieved ~80\% of the ideal throughput while training a 50 billion parameter decoder model using model parallelism, and ~70\% per GPU FLOPS compared to a single VM with the High-Performance Linpack benchmark.},
booktitle = {Proceedings of the 30th ACM International Conference on Architectural Support for Programming Languages and Operating Systems, Volume 2},
pages = {1348–1364},
numpages = {17},
keywords = {ai infrastructure, cloud computing, high performance computing, virtualization},
location = {Rotterdam, Netherlands},
series = {ASPLOS '25}
}
```

```
Corresponding Authors:

Apoorve Mohan: apoorve.mohan@ibm.com
Seetharami Seelam: sseelam@us.ibm.com
```


