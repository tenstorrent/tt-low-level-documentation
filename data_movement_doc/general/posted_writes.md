# Posted NOC Write Transactions

## Overview

TT-Metal uses non-posted writes as the default for all NOC write transactions. This document describes the benefits and caveats of using posted writes as an alternative approach for improved performance in specific scenarios.

## Background

### Non-Posted vs Posted Writes

**Non-Posted Writes (Default)**
- Each transaction includes a response acknowledgment (ACK) message
- Provides confirmation that data has been successfully written to the destination
- Guarantees completion before the transaction is considered finished
- Uses the `NOC_CMD_RESP_MARKED` flag in the NOC command field

**Posted Writes**
- Do not require an acknowledgment message
- Transaction is considered complete once it leaves the source core
- No confirmation of successful destination write
- Does not set the `NOC_CMD_RESP_MARKED` flag

## When to Use Posted Writes

Posted writes are beneficial when you only need to ensure that:
1. The transaction has departed from the current core
2. The relevant L1 address can be reused/modified
3. You don't need confirmation of successful destination write completion

## Performance Benefits

Posted writes can provide significant performance improvements by:

1. **Reducing ACK Traffic Congestion**: Multiple NOC writes can cause congestion in acknowledgment traffic, leading to performance degradation when using barriers.

2. **Eliminating Round-Trip Latency**: By using posted barriers (sent/flushed registers) for synchronization instead of waiting for ACKs, you can skip the round-trip latency of acknowledgment messages.

### Measured Performance Gains

#### NOC-Heavy Operations
| Operation                         | Performance Gain |
|-----------------------------------|------------------|
| Interleaved to sharded            | Up to 50%        |
| Reshard (NOC bandwidth saturated) | Up to 60%        |
| Untilize                          | Up to 75%        |
| Tilize                            | ~2% (negligible) |
| Transpose                         | ~2% (negligible) |
| Concat                            | ~5% (negligible) |

#### Data Movement Primitives
- **One-to-one**: Smaller transactions (non-NOC-bandwidth-saturating) see significant gains; larger transactions show minimal improvement
- **One-to-all**: Similar pattern to one-to-one operations

## Technical Implementation

### API Usage

Most NOC write functions support posted writes through template parameters:

```cpp
// Example: Using posted writes with template parameter
noc_write_with_state<DM_DEDICATED_NOC, cmd_buf, flags, CQ_NOC_SEND, CQ_NOC_WAIT, true, true>(
    noc, src_addr, dst_addr, size);
//                                                                           ^^^^
//                                                                        posted = true
```

### Barrier Functions

Different barrier functions are available depending on the write type:

```cpp
// For non-posted writes - waits for completion (ACK received)
noc_async_write_barrier(noc);

// For non-posted writes - waits for departure only (sent but not ACKed)
noc_async_writes_flushed(noc);

// For posted writes - waits for departure only
noc_async_posted_writes_flushed(noc);
```

### Counter Types

The system tracks different counter types for monitoring:
- `NONPOSTED_WRITES_NUM_ISSUED`: Number of non-posted writes issued
- `NONPOSTED_WRITES_ACKED`: Number of non-posted writes acknowledged
- `POSTED_WRITES_NUM_ISSUED`: Number of posted writes issued

## Caveats and Risk Considerations

⚠️ **IMPORTANT**: Posted writes should be used at your own risk and require careful consideration of synchronization requirements.

### Primary Risk: Data Race Conditions

Posted writes provide **no guarantee** about when data is actually written to the destination. This can create data and synchronization hazards in the following scenarios:

1. **Multi-kernel synchronization**: When multiple data movement kernels need to coordinate on moving data
2. **Compute-data dependencies**: When compute operations depend on data that may not have arrived yet

### Common Data Race Example

```
Timeline: Producer A → Consumer C (via signal) → Reader C

1. Producer A writes data to core B using posted write
2. Producer A immediately signals/increments semaphore to core C
3. Core C receives signal and starts reading from core B
4. PROBLEM: Data may not have arrived at core B yet
5. Core C reads garbage/stale values
```

**Root Cause**: The semaphore increment happens before the data write is fully committed to core B's memory.

### Best Practices for Safe Usage

1. **Use appropriate barriers**: Employ `noc_async_posted_writes_flushed()` when you need to ensure posted writes have at least departed
2. **Careful synchronization design**: Ensure your synchronization protocol accounts for the lack of write completion guarantees
3. **Profile thoroughly**: Measure performance gains in your specific use case to ensure the risk is justified
4. **Consider hybrid approaches**: Use posted writes for non-critical data paths while maintaining non-posted writes for critical synchronization points

## Conclusion

Posted NOC writes can provide substantial performance improvements (up to 75% in some cases) by eliminating acknowledgment traffic and round-trip latencies. However, they require careful consideration of data dependencies and synchronization requirements to avoid race conditions. Use posted writes when the performance benefits outweigh the additional complexity of ensuring proper data ordering and synchronization.