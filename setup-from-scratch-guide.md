# Setup From Scratch Guide

This document provides a step-by-step tutorial for people who wish to build and run Optimus on their own compatible HARP platforms.

Deploying Optimus on a compatible system requires three steps: 1) choosing a set of accelerators and synthesizing them to a bitstream, 2) installing the host software, and 3) booting the guest and installing the guest software.

### 3.1 Synthesizing Accelerators

After choosing a desired set of accelerators, we need to synthesize these accelerators to a *bitstream* for deployment. This section provides a tutorial of how to synthesize a set of accelerators onto an FPGA, as well as the configuration files we used in our evaluation.

Quartus Pro 17.0 and Blue Bitstream (BBS) SR-6.4.0 are used to synthesize the accelerators and generate the bitstreams. The former requires a commercial license, and the latter is provided together with the hardware platform. Generation of a single bitstream may take 2 to 10 hours, depending on the computing power and accelerator configuration.

First, add the following lines to `.bashrc` or `.zshrc`:

```
export QUARTUS_HOME=/path/to/quartus
export PATH=$QUARTUS_HOME/bin:$PATH
export FPGA_BBB_CCI_SRC=/path/to/optimus-intel-fpga-bbb
export OPAE_PLATFORM_ROOT=/path/to/BBS_6.4.0
export PATH=$OPAE_PLATFORM_ROOT/bin:$PATH                                       
export LM_LICENSE_FILE=/path/to/quartus/license
```

In order to synthesize different accelerators into one bitstream, we need to create a *top level design*, which connects all the accelerators to the hardware monitor. We can then use the standard HARP synthesis scripts to perform the synthesis.

We provide configurations for our benchmarks in `optimus-intel-fpga-bbb/samples/tutorial/synth_config`. We use a configuration with 8 MemBench accelerator as an example. Other configurations cna be synthesized using a similar process to this tutorial.

For MemBench, note that you may need to edit `membench.txt` to fill in the correct pathname of the source code. In this example, the top level design is `cci_mux.sv`.

```
cd /path/to/optimus-intel-fpga-bbb/samples/tutorial/synth_config/bbb_8mux
afu_synth_setup -s membench.txt build_membench_8mux
```

This will create a folder in the current directory named "`build_membench_8mux`", which contains the files needed to synthesize the design. You can begin synthesis (2-10 hours) with the following:

```
cd build_membench_8mux
$OPAE_PLATFORM_ROOT/bin/run.sh
```

After synthesis finishes, you will see a file named "`cci_mux.gbs`" in the directory, which is the generated bitstream.

### 3.2 Optimus Installation and Usage

Given a set of bitstreams, we can install the host software, configure the FPGA with a bitstream, and boot virtual machines. The host software includes the Optimus Hypervisor and some user space tools. The Optimus Hypervisor is implemented as a Linux kernel module, which is needed to configure and virtualize the FPGA. The user space tools are used to manage the FPGA from the host.

The first step is to installing necessary dependencies.

```
yum update
yum groupinstall 'Development Tools'
yum install kernel-devel cmake libuuid-devel json-c-devel qemu
```

Compile and install the host OPAE library, which contains all the user space tools we need to perform the configuration and management.

```
git clone https://github.com/efeslab/optimus-opae-sdk
cd optimus-opae-sdk
mkdir build; cd build
cmake ..
make -j10
make install
```

Clone and compile the host kernel module.

```
git clone https://github.com/efeslab/optimus-host-module
cd optimus-host-module
make
./insdrv.sh # use this to load the driver
```

You can then load a bitstream and reload the driver.

```
sudo /usr/local/bin/fpgaconf -b 0x5e your_benchmark.gbs
./reload.sh # For now, you must do this everytime you change the bitstream
```

To print information about the bitstream, use the following command:

```
cat /sys/class/fpga/intel-fpga-dev.0/intel-fpga-port.0/optimus/info
```

To boot a guest VM, you first need a virtual device. We use the standard *vfio-mdev* interface to create a virtual device. In the example below, we use Accelerator 0 from the bitstream.

```
export VAI_UUID=`uuidgen`
sudo su -c "echo $VAI_UUID > /sys/class/fpga/intel-fpga-dev.0/intel-fpga-port.0/mdev_supported_types/intel-fpga-port-direct-0/create"
```

Then, allocate huge pages for the guest.

```
echo 40000 | sudo tee /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
```

To run QEMU with the virtual device:

```
sudo qemu-system-x86_64 \
    -enable-kvm \
    -smp 4 \
    -m 10000 \
    -mem-path /dev/hugepages \
    -mem-prealloc \
    -hda /home/jcma/img/vm0.qcow2 \
    -vnc :10 \
    -device vfio-pci,sysfsdev=/sys/bus/mdev/devices/$VAI_UUID \
    -serial stdio
```

With QEMU running, you can now connect to the VM and run the desired applications.

### 3.3 Guest Driver and Library Installation Guide

The last step for deploying Optimus is to the install guest driver and libraries. The driver and libraries communicate with the virtualized hardware, and additionally provide a simple interface to software using the accelerator.

The first step is to install necessary dependencies in the guest.

```
yum update
yum groupinstall 'Development Tools'
yum install kernel-devel cmake libuuid-devel json-c-devel
```

Reboot the guest to use the modified kernel. The name of the driver is "VAI", which is abbreviated from "virtual accelerator interface". The command `depmod` registers the kernel module, so the driver will be loaded automatically after a virtual accelerator is found.

```
git clone https://github.com/efeslab/optimus-guest-driver
cd optimus-guest-driver
make
cp vai.ko /lib/modules/`uname -r`/extra
depmod
```

Next, we need to install the guest libraries. Since we modify the Intel's OPAE SDK to support VAI, this will also build the original OPAE libraries.

```
git clone https://github.com/efeslab/optimus-opae-sdk
# the default branch should be vai, do not modify it
cd optimus-opae-sdk
mkdir build; cd build
cmake ..
make -j10
make install
```

Finally, we need to install the software part of the MPF library. In Optimus, virtual addressing is provided by the hypervisor, so the VTP functionality in MPF is disabled. However, MPF can still provide features such as response ordering.

```
git clone https://github.com/efeslab/optimus-intel-fpga-bbb
cd optimus-intel-fpga-bbb/BBB_cci_mpf/sw
mkdir build; cd build
cmake ..
make -j10
make install
```

You can now run any desired applications in the guest.