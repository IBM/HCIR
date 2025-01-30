## Overview
 
This repository contains the publically released artifacts for the paper titled `Vela: A Virtualized LLM Training System with GPU Direct RoCE` to be published at the 2025 International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS'25). 

```
Following is the list of the released artifcations:

- Details for provisioning GPU Direct RoCE capable VMs on a Linux-KVM-QEMU-based based hypervisor optimized of LLM Training workloads (see "vela_asplos2025/vmsetup").

- Pytorch-based benchmarks for the commonly used collective communication operations employed in LLM training workloads. The key communication collective benchmarks are `allreduce-loop.py`, `allgather-loop.py`, and `reduce-scatter.py` (see "vela_asplos2025/pycommbench").

- Code to replicate the potential RAS issue due to passthrough device fuzzing (see "vela_asplos2025/ras").
```


```
Corresponding Authors:

Apoorve Mohan: apoorve.mohan@ibm.com
Seetharami Seelam: sseelam@us.ibm.com
```


