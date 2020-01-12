# Pass-Through Setup Guide

In our paper, we compared the performance of different accelerators under Optimus against the performance under pass-through based virtualization. Here, we give a brief guide of setting up the pass-through environment.

### 1. Software Installation

We modified the original OPAE driver/library, using the IOMMU to perform virtual-to-physical address translation (instead of using the MPF hardware library).

In order to run the pass-through experiments, we need to install the following components with specific branches:

- optimus-opae-sdk (branch: `iommu`)
- optimus-host-module (branch: `pviommu`)
- optimus-intel-fpga-bbb (brach: `iommu`)
- hardcloud_no_openmp (branch: `iommu`)

### 2. Hardware Synthesis

While synthesizing the hardware, you will need to use `nomux.txt` under the accelerator directory (e.g., `optimus-intel-fpga-bbb/sample/tutorial/vai_membench/hw/nomux.txt`), instead of the one under the directory `synth_config`.

### 3. Assign Device and Boot Virtual Machines

Currently, pass-through does not support using partial reconfiguration inside a VM. As a result, we must reconfigure the FPGA to our desired accelerator before boot the virtual machine.

We provide some scripts to help device assignment [here](https://github.com/efeslab/optimus-scripts/tree/master/pt).