# Data Movement Performance Plots

This file contains visual representations of performance results from our various data movement primitive tests. A separate set of results is present for both architectures (i.e. Wormhole_B0 and Blackhole).

In all plots, "Sender" or "Writer" refers to outgoing data movement primitives where the master core(s) issue(s) a write transaction to subordinate cores or to DRAM. RISCV 0/NOC 0 is typically used for these transactions. On the other hand, "Receiver" or "Reader" refers to incoming data movement primitives where the master core(s) issue(s) a read transaction. RISCV 1/NOC 1 is typically used for these transactions.

For each test case, two plots are given to provide meaning from both the Hardware and the Software perspectives. The "Transaction Size vs Duration" plot depicts how long data movement takes to complete for a specific combination of test parameters (i.e. number of transactions and transaction sizes). This may be useful for kernel developers to gauge the latency of their OPs. The "Transaction Size vs Bandwidth" plot depicts how much the NOC capacity is saturated by data movement kernels using different combination of test parameters. This may be useful to gauge the effective performance of different data movement scenarios.

For steps to reproduce these plots, refer to our general [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/README.md).

For more information on each test, refer to the README under each test primitive directory, e.g. [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/dram_unary/README.md) for DRAM tests

## Wormhole_B0

### DRAM Interleaved Packet Sizes

![DRAM Interleaved Packet Sizes](./wormhole_b0/images/DRAM%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/DRAM%20Interleaved%20Packet%20Sizes.csv).

Dram read bandwidth saturates at about 37 B/cycle, according to HW experiments. DRAM write bandwidth should saturate at 64 B/cycle, instead of 35 B/c. There may be some configuration problem with the dram controller/phy or this may be the physical limit of the dram.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/dram_unary/README.md).

### One to One Packet Sizes

![One to One Packet Sizes](./wormhole_b0/images/One%20to%20One%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20One%20Packet%20Sizes.csv).

Bandwidth in steady state, with > 2KB packet sizes, is close to theoretical max. Under 2KB, the bandwidth is limitted by either the RISC latency or by the NOC sending from L1 latency.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_one/README.md).

### Loopback Packet Sizes

![Loopback Packet Sizes](./wormhole_b0/images/Loopback%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/Loopback%20Packet%20Sizes.csv).

Loopback will have similar characteristics to the one to one test, however it uses two ports to send and receive data, as such it is more likely to cause contention.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/loopback/README.md).

### One from One Packet Sizes

![One from One Packet Sizes](./wormhole_b0/images/One%20from%20One%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20from%20One%20Packet%20Sizes.csv).

Bandwidth in steady state, with > 2KB packet sizes, is close to theoretical max. Under 2KB, the bandwidth is limitted by the RISC latency.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_from_one/README.md).

### One to All Packet Sizes
#### Unicast
##### 2x2
##### With Loopback

![One to All Unicast 2x2 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Unicast%202x2%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Unicast%202x2%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Unicast 2x2 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Unicast%202x2%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Unicast%202x2%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a small grid. The bandwidth characteristics are similar to the one to one test. Note that it may appear that multicast has lower bandwidth, however multicast sends less data and has much lower latency, so it is prefered to use multicast.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 4x4
##### With Loopback

![One to All Unicast 4x4 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Unicast%204x4%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Unicast%204x4%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Unicast 4x4 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Unicast%204x4%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Unicast%204x4%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a medium grid. The bandwidth characteristics are similar to the one to one test. As the grid size increases, the number of transactions needed to saturate NOC decreases because the NOC needs to send num cores more packets. Note that it may appear that multicast has lower bandwidth, however multicast sends less data and has much lower latency, so it is prefered to use multicast.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 10x10
##### With Loopback

![One to All Unicast 10x10 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Unicast%2010x10%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Unicast%2010x10%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Unicast 10x10 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Unicast%2010x10%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Unicast%2010x10%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a large grid. The bandwidth characteristics are similar to the one to one test. As the grid size increases, the number of transactions needed to saturate NOC decreases because the NOC needs to send num cores more packets. Note that it may appear that multicast has lower bandwidth, however multicast sends less data and has much lower latency, so it is prefered to use multicast.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

#### Multicast Unlinked
##### 2x2
##### With Loopback

![One to All Multicast Unlinked 2x2 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%202x2%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%202x2%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Unlinked 2x2 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%202x2%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%202x2%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a small grid using unlinked multicast. Bandwidth degrades due to path reserve being done after every transaction.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 5x5
##### With Loopback

![One to All Multicast Unlinked 5x5 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%205x5%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%205x5%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Unlinked 5x5 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%205x5%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%205x5%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a medium grid using unlinked multicast. Bandwidth degrades due to path reserve being done after every transaction. As the grid size increases, the number of write acks increases which degrades bandwidth.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 11x10
##### With Loopback

![One to All Multicast Unlinked 11x10 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%2011x10%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%2011x10%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Unlinked 11x10 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%2011x10%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%2011x10%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a large grid using unlinked multicast. Bandwidth degrades due to path reserve being done after every transaction. As the grid size increases, the number of write acks increases which degrades bandwidth.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

#### Multicast Linked
##### 2x2
##### With Loopback

![One to All Multicast Linked 2x2 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%202x2%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%202x2%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked 2x2 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%202x2%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%202x2%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a small grid using linked multicast. Linked causes path reserve to be done only once for all transactions, as such performance approaches theoretical.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 5x5
##### With Loopback

![One to All Multicast Linked 5x5 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%205x5%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%205x5%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked 5x5 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%205x5%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%205x5%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a medium grid using linked multicast. Linked causes path reserve to be done only once for all transactions, as such performance approaches theoretical. As the grid size increases, the number of write acks increases which degrades bandwidth. Posted multicasts do not have this issue, however it is not safe to use posted multicast due to a hardware bug.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 11x10
##### With Loopback

![One to All Multicast Linked 11x10 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%2011x10%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%2011x10%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked 11x10 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%2011x10%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%2011x10%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a large grid using linked multicast. Linked causes path reserve to be done only once for all transactions, as such performance approaches theoretical. As the grid size increases, the number of write acks increases which degrades bandwidth. Posted multicasts do not have this issue, however it is not safe to use posted multicast due to a hardware bug.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

#### Multicast Linked Semaphore
##### 2x2
##### With Loopback

![One to All Multicast Linked Semaphore 2x2 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%20Semaphore%202x2%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%202x2%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked Semaphore 2x2 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%20Semaphore%202x2%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%202x2%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a small grid using linked multicast with semaphore synchronization.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 5x5
##### With Loopback

![One to All Multicast Linked Semaphore 5x5 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%20Semaphore%205x5%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%205x5%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked Semaphore 5x5 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%20Semaphore%205x5%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%205x5%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a medium grid using linked multicast with semaphore synchronization.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 11x10
##### With Loopback

![One to All Multicast Linked Semaphore 11x10 Packet Sizes with Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%20Semaphore%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked Semaphore 11x10 Packet Sizes without Loopback](./wormhole_b0/images/One%20to%20All%20Multicast%20Linked%20Semaphore%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a large grid using linked multicast with semaphore synchronization.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

### One from All Packet Sizes

![One from All Packet Sizes](./wormhole_b0/images/One%20from%20All%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/One%20from%20All%20Packet%20Sizes.csv).

At small packet sizes, the bandwidth is limited by the RISC latency. As the packet size increases, the bandwidth approaches 64 B/cycle. Similar to the one from one test.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_from_all/README.md).

### All to All Packet Sizes

![All to All Packet Sizes](./wormhole_b0/images/All%20to%20All%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/All%20to%20All%20Packet%20Sizes.csv).

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/all_to_all/README.md).

### All from All Packet Sizes

![All from All Packet Sizes](./wormhole_b0/images/All%20from%20All%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./wormhole_b0/csv/All%20from%20All%20Packet%20Sizes.csv).

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/all_from_all/README.md).

## Blackhole

### DRAM Interleaved Packet Sizes

![DRAM Interleaved Packet Sizes](./blackhole/images/DRAM%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/DRAM%20Interleaved%20Packet%20Sizes.csv).

Dram read bandwidth saturates at about 37 B/cycle, according to HW experiments. DRAM write bandwidth should saturate at 64 B/cycle, instead of 35 B/c. There may be some configuration problem with the dram controller/phy or this may be the physical limit of the dram.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/dram_unary/README.md).

### One to One Packet Sizes

![One to One Packet Sizes](./blackhole/images/One%20to%20One%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20One%20Packet%20Sizes.csv).

Bandwidth in steady state, with > 2KB packet sizes, is close to theoretical max. Under 2KB, the bandwidth is limitted by either the RISC latency or by the NOC sending from L1 latency.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_one/README.md).

### Loopback Packet Sizes

![Loopback Packet Sizes](./blackhole/images/Loopback%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/Loopback%20Packet%20Sizes.csv).

Loopback will have similar characteristics to the one to one test, however it uses two ports to send and receive data, as such it is more likely to cause contention.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/loopback/README.md).

### One from One Packet Sizes

![One from One Packet Sizes](./blackhole/images/One%20from%20One%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20from%20One%20Packet%20Sizes.csv).

Bandwidth in steady state, with > 2KB packet sizes, is close to theoretical max. Under 2KB, the bandwidth is limitted by the RISC latency.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_from_one/README.md).

### One to All Packet Sizes
#### Unicast
##### 2x2
##### With Loopback

![One to All Unicast 2x2 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Unicast%202x2%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Unicast%202x2%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Unicast 2x2 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Unicast%202x2%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Unicast%202x2%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a small grid. The bandwidth characteristics are similar to the one to one test. Note that it may appear that multicast has lower bandwidth, however multicast sends less data and has much lower latency, so it is prefered to use multicast.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 4x4
##### With Loopback

![One to All Unicast 4x4 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Unicast%204x4%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Unicast%204x4%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Unicast 4x4 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Unicast%204x4%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Unicast%204x4%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a medium grid. The bandwidth characteristics are similar to the one to one test. As the grid size increases, the number of transactions needed to saturate NOC decreases because the NOC needs to send num cores more packets. Note that it may appear that multicast has lower bandwidth, however multicast sends less data and has much lower latency, so it is prefered to use multicast.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 10x10
##### With Loopback

![One to All Unicast 10x10 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Unicast%2010x10%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Unicast%2010x10%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Unicast 10x10 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Unicast%2010x10%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Unicast%2010x10%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a large grid. The bandwidth characteristics are similar to the one to one test. As the grid size increases, the number of transactions needed to saturate NOC decreases because the NOC needs to send num cores more packets. Note that it may appear that multicast has lower bandwidth, however multicast sends less data and has much lower latency, so it is prefered to use multicast.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

#### Multicast Unlinked
##### 2x2
##### With Loopback

![One to All Multicast Unlinked 2x2 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Multicast%202x2%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%202x2%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Unlinked 2x2 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Multicast%202x2%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%202x2%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a small grid using unlinked multicast. Bandwidth degrades due to path reserve being done after every transaction.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 5x5
##### With Loopback

![One to All Multicast Unlinked 5x5 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Multicast%205x5%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%205x5%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Unlinked 5x5 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Multicast%205x5%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%205x5%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a medium grid using unlinked multicast. Bandwidth degrades due to path reserve being done after every transaction. As the grid size increases, the number of write acks increases which degrades bandwidth.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 11x10
##### With Loopback

![One to All Multicast Unlinked 11x10 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Multicast%2011x10%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%2011x10%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Unlinked 11x10 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Multicast%2011x10%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%2011x10%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a large grid using unlinked multicast. Bandwidth degrades due to path reserve being done after every transaction. As the grid size increases, the number of write acks increases which degrades bandwidth.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

#### Multicast Linked
##### 2x2
##### With Loopback

![One to All Multicast Linked 2x2 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%202x2%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%202x2%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked 2x2 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%202x2%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%202x2%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a small grid using linked multicast. Linked causes path reserve to be done only once for all transactions, as such performance approaches theoretical.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 5x5
##### With Loopback

![One to All Multicast Linked 5x5 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%205x5%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%205x5%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked 5x5 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%205x5%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%205x5%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a medium grid using linked multicast. Linked causes path reserve to be done only once for all transactions, as such performance approaches theoretical. As the grid size increases, the number of write acks increases which degrades bandwidth. Posted multicasts do not have this issue, however it is not safe to use posted multicast due to a hardware bug.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 11x10
##### With Loopback

![One to All Multicast Linked 11x10 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%2011x10%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%2011x10%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked 11x10 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%2011x10%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%2011x10%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a large grid using linked multicast. Linked causes path reserve to be done only once for all transactions, as such performance approaches theoretical. As the grid size increases, the number of write acks increases which degrades bandwidth. Posted multicasts do not have this issue, however it is not safe to use posted multicast due to a hardware bug.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

#### Multicast Linked Semaphore
##### 2x2
##### With Loopback

![One to All Multicast Linked Semaphore 2x2 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%20Semaphore%202x2%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%202x2%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked Semaphore 2x2 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%20Semaphore%202x2%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%202x2%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a small grid using linked multicast with semaphore synchronization.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 5x5
##### With Loopback

![One to All Multicast Linked Semaphore 5x5 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%20Semaphore%205x5%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%205x5%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked Semaphore 5x5 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%20Semaphore%205x5%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%205x5%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a medium grid using linked multicast with semaphore synchronization.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

##### 11x10
##### With Loopback

![One to All Multicast Linked Semaphore 11x10 Packet Sizes with Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%20Semaphore%20Packet%20Sizes%20with%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%20Packet%20Sizes%20with%20Loopback.csv).

##### Without Loopback

![One to All Multicast Linked Semaphore 11x10 Packet Sizes without Loopback](./blackhole/images/One%20to%20All%20Multicast%20Linked%20Semaphore%20Packet%20Sizes%20without%20Loopback.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20to%20All%20Multicast%20Linked%20Semaphore%20Packet%20Sizes%20without%20Loopback.csv).

This test sends to a large grid using linked multicast with semaphore synchronization.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_to_all/README.md).

### One from All Packet Sizes

![One from All Packet Sizes](./blackhole/images/One%20from%20All%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/One%20from%20All%20Packet%20Sizes.csv).

At small packet sizes, the bandwidth is limited by the RISC latency. As the packet size increases, the bandwidth approaches 64 B/cycle. Similar to the one from one test.

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/one_from_all/README.md).

### All to All Packet Sizes

![All to All Packet Sizes](./blackhole/images/All%20to%20All%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/All%20to%20All%20Packet%20Sizes.csv).

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/all_to_all/README.md).

### All from All Packet Sizes

![All from All Packet Sizes](./blackhole/images/All%20from%20All%20Packet%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./blackhole/csv/All%20from%20All%20Packet%20Sizes.csv).

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/all_from_all/README.md).

## Quasar

> Results collected on the `emu-quasar-1x3` emulator (test id 912, single DM core).
> Each timed region repeats the write(+flush) **100 times** inside one profiler zone and
> the reported value is the zone duration divided by the iteration count, so fixed
> per-run overhead is amortized and the numbers reflect steady-state per-write cost.
> Because the 1x3 emulator has no fast-dispatch cores, runs execute in slow-dispatch
> mode, under which the profiler does not increment `run_host_id`; the plotted per-run
> values are reconstructed from the profiler CSV in execution order.

### Quasar Cache Write Sizes

Compares single-DM-core write performance to Tensix L1, swept over total data size
(1B–2KB), across three write modes:

- **Uncached (1B)** — uncached port (`base + MEM_L1_UNCACHED_BASE`, +4MB alias), byte-at-a-time stores.
- **Uncached (8B)** — same uncached port, but 64-bit stores (the natural DM-core store width).
- **Cached+Flush (8B)** — 64-bit cacheable stores then `flush_l2_cache_range` (`ceil(N/64)` 64B line flushes).

Key takeaways (amortized cycles per write):

- **Store width dominates the uncached port.** With 8-byte stores the uncached port costs ~13–14 cycles per store (e.g. 64B in ~109 cycles, 2KB in ~3.4k); byte-at-a-time is ~8x worse (2KB in ~24.3k) since each byte pays full TL1 latency. Below 8B the two uncached modes coincide (a sub-8B write is a byte write either way).
- **`flush_l2_cache_range` has a fixed ~160-cycle floor.** `Cached+Flush (8B)` is essentially flat at ~160–190 cycles up to one cache line (64B), then grows with `ceil(N/64)` line flushes. It is slower than plain `Uncached (8B)` across the whole range for this write-only, no-reuse pattern; it would only pay off with subsequent reads or scattered sub-line updates.
- Top row is duration (amortized cycles/write); bottom row is bandwidth (bytes/cycle); the right column zooms into the 0–64B region. The step in `Uncached (8B)` at 8B is where real 64-bit stores replace the byte tail.

**When to use which (write-only to L1):**

- **Any size, fire-and-forget writes → uncached port with 8-byte stores.** It is fastest at every size in this sweep; cache+flush is never faster.
- **< 8 bytes → uncached** (store width is irrelevant below 8B). Cache+flush costs its ~160-cycle flush floor, 3–10x worse.
- **Cache + `flush_l2_cache_range` → only with reuse.** It pays off when the written data is subsequently *read back* by the core, or when many scattered sub-64B updates coalesce in cache before a single flush. This write-only, no-reuse test does not exercise those cases, so it shows cache+flush purely as overhead.
- The intuitive "cached beats uncached above some size" only holds against *byte-granular* uncached writes (Uncached-1B loses to cache+flush above ~16B); wide (8B) uncached beats both everywhere, so the real lesson is to use 8-byte stores.

> Note: an earlier version of this test wrote byte-at-a-time and timed a single un-amortized pass, which suggested cache+flush was ~2x faster. Both were artifacts (byte-granular writes + a ~325-cycle fixed measurement overhead). With 8-byte stores and 100-iteration amortization the numbers align with prior standalone measurements (uncached ~15 cyc/store, cache+flush ~160-cycle flush floor).

![Quasar Cache Write Sizes](./quasar/images/Quasar%20Cache%20Write%20Sizes.png)
To view these results in a table, refer to the relevant [csv](./quasar/csv/Quasar%20Cache%20Write%20Sizes.csv).

For more information on this primitive, refer to [README](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement/quasar_cache_perf/README.md).
