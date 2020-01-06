# Preemption Interface

This document provides a description of Optimus's preemption interface. The accelerator needs to implement this interface to support preemptable temporal multiplexing.

**Note:** In the current implementation of Optimus, the hypervisor is only responsible for sending the preempt command to the accelerator. The guest application is responsible for setting up the preemption buffer, checking job status, and resuming the job after it is preempted.

### 1. Snapshot Buffer

The guest application needs to allocate a buffer, which is called the *snapshot buffer*, to store the context of the accelerator. The buffer lies in guest user address space, and should be allocated via the memory allocation API provided by the guest library.

### 2. Control Register Definition

The interface is defined as 3 MMIO registers: *CSR_TRANSACTION_CTL*, *CSR_STATE_SIZE_PG*, and *CSR_SNAPSHOT_ADDR*.

**CSR_TRANSACTION_CTL:**

- Offset: 0x18
- Read from this register should return the status of the accelerator, which includes:
  - tsIDLE(0): the accelerator is idle
  - tsRUNNING(1): the accelerator is running a task
  - tsFINISH(2): the acceleration task on this accelerator is finished
  - tsPAUSED(4): the acceleration task on this accelerator is paused and preempted out
- Write the following values to this register should causing the following behaviors:
  - tsctlSTART_NEW(1): start a new task on this accelerator
  - tsctlSTART_RESUME(5): read the accelerator context from the snapshot buffer and restore the acceleration job
  - tsctlPAUSE(6): preempt the accelerator and save the context to the snapshot buffer

**CSR_STATE_SIZE_PG:**

* Offset: 0x20
* User application should read this register to get the number of 4K pages of the snapshot buffer.

**CSR_SNAPSHOT_ADDR:**

* Offset: 0x28
* After the snapshot buffer is allocated, the user application should write the cacheline address of the buffer to this register.

### 3. Examples

Among our benchmarks, MemBench and LinkedList are implemented with preemption support. You may use them as examples. You can find MemBench and LinkedList at https://github.com/efeslab/optimus-intel-fpga-bbb/tree/master/samples/tutorial.