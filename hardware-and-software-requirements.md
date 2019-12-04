## Hardware and Software Requirements

Optimus requires the following hardware:

- Intel HARP platform (with Skylake CPUs, Arria 10 FPGAs, and Blue Bitstream version SR-6.4.0).
- Intel VT-d (must be enabled in the BIOS).

Optimus requires the following software, some of which is proprietary:

- GCC 4.8 (for kernel modules and libraries).
- Quartus Prime Pro 17.0.0 (for FPGA synthesis).
- CentOS 7.5 with Linux kernel version 5.1.0-rc6. When compiling the kernel, the configuration option FPGA_DFL must be disabled.
- QEMU 3.0.1.