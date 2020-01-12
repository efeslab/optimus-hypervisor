# Optimus: A Hypervisor for Shared-Memory FPGA Platforms

All of the source code for Optimus is open source, and is comprised of the following components:

**Optimus Hypervisor**

- https://github.com/efeslab/optimus-host-module
- The Optimus Hypervisor is provided as a host kernel module.

**Hardware Monitor**

- https://github.com/efeslab/optimus-intel-fpga-bbb/tree/master/BBB_vai_mux_nested/hw
- The hardware monitor is provided as a basic building block (BBB).

**Guest Driver**

- https://github.com/efeslab/optimus-guest-driver
- The guest driver is provided as a guest kernel module, which drives the virtualized accelerator.

**Guest Core Library**

- https://github.com/efeslab/optimus-opae-sdk
- The guest core library is embedded in the Intel OPAE SDK.

**Guest MPF Library**

- https://github.com/efeslab/optimus-intel-fpga-bbb/tree/master/BBB_cci_mpf/sw
- MPF is a software/hardware library which provides a group of memory properties such as memory ordering. We have modified the library to support virtual accelerators.

**Benchmarks**

- MemBench, LinkedList, SSSP, and Bitcoin can be found in https://github.com/efeslab/optimus-intel-fpga-bbb/tree/master/samples/tutorial. We additionally include our ports of three benchmarks from HardCloud (grayscale, sobel, and gaussian) in this repository, as these benchmarks required significant changes to run on our platform.
- The hardware portion of the remaining HardCloud benchmarks can be found in https://github.com/efeslab/hardcloud. The software portion of these benchmarks can be found in https://github.com/efeslab/hardcloud_no_openmp. Note that the benchmarks are compiled without OpenMP support.
- The synthesis configuration for these benchmarks can be found in https://github.com/efeslab/optimus-intel-fpga-bbb/tree/master/samples/tutorial/synth_config.



**Setup Guide**

- Our code is developed and tested using the hardware and software specificed [here](hardware-and-software-requirements.md).
- A step-by-step tutorial is available [here](setup-from-scratch-guide.md) to help you build everything from scratch.
- Some scripts we used in our experiment can be found [here](https://github.com/efeslab/optimus-scripts).
- The definition of Optimus's preemption interface can be found [here](https://github.com/efeslab/optimus-hypervisor/blob/master/preemption-interface.md).

