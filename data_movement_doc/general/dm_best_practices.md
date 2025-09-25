# Data Movement Performance Guide for Operation Writers

This guide focuses on performance optimization and common pitfalls when using TT-Metal's data movement APIs. Rather than just explaining what each API does, we'll cover **why** certain patterns perform better and **what mistakes** to avoid.

## Critical Performance Mistakes to Avoid

### 1. **Missing or Misplaced Barriers**

**The Problem:** NOC operations are asynchronous. Without proper barriers, your kernel will continue executing while data is still in flight, leading to race conditions, corrupted data, or complete hangs.

```cpp
// WRONG - No barrier, data might not be ready
noc_async_read(src_addr, dst_addr, size);
process_data(dst_addr);  // BUG: Data might not be there yet!

// WRONG - Barrier after every operation kills performance
for (uint32_t i = 0; i < num_tiles; i++) {
    noc_async_read(src_addr + i * tile_size, dst_addr, tile_size);
    noc_async_read_barrier();  // BAD: Destroys async benefits
    process_tile(dst_addr);
}

// CORRECT - Batch operations, then barrier
constexpr uint32_t batch_size = 4;  // Don't saturate NOC
for (uint32_t batch = 0; batch < num_tiles; batch += batch_size) {
    uint32_t tiles_in_batch = std::min(batch_size, num_tiles - batch);
    
    // Issue batch of async reads
    for (uint32_t i = 0; i < tiles_in_batch; i++) {
        noc_async_read(src_addr + (batch + i) * tile_size, 
                      dst_addr + i * tile_size, tile_size);
    }
    
    // Single barrier for the batch
    noc_async_read_barrier();
    
    // Process all tiles in the batch
    for (uint32_t i = 0; i < tiles_in_batch; i++) {
        process_tile(dst_addr + i * tile_size);
    }
}
```

**When barriers are critical:**
- After a batch of `noc_async_read` operations before using the data
- Before kernel termination if you have outstanding writes
- Before changing NOC state or switching contexts
- When NOC command buffers are getting full. This means that the buffers are at capacity, they can't accept any more commands 
  right now. If the buffer is full, the code enters a waiting loop until space becomes available. This creates backpressure in the system.

**Performance tips:** 
- Batch multiple operations before using barriers to maximize async benefits
- Use `noc_async_writes_flushed()` instead of `noc_async_write_barrier()` when you only need to ensure writes are sent (not completed)

### 2. **Circular Buffer Mismanagement**

**The Problem:** CB operations must be paired and sized correctly. 

```cpp
// WRONG - Mismatched reserve/push
cb_reserve_back(cb_id, 2);
uint32_t addr = get_write_ptr(cb_id);
// ... write data ...
cb_push_back(cb_id, 1);  // BUG: Reserved 2, pushed 1!

// CORRECT - Perfect pairing and proper sizing
cb_reserve_back(cb_id, 1);
uint32_t addr = get_write_ptr(cb_id);
noc_async_read(src_addr, addr, size);
noc_async_read_barrier();  // Barrier when data is needed
cb_push_back(cb_id, 1);  // Matches reserve
```

**Critical CB rules:**
- `cb_reserve_back(cb_id, n)` must be followed by one or more calls to `cb_push_back(cb_id, y)` 
  such that the total number of elements pushed equals `n` (i.e., `n = x * y`, where `x` 
  is the number of `cb_push_back` calls and `y` is the number of elements per call).
- The same principle applies to `cb_wait_front(n)` and `cb_pop_front(n)`.
- CB size must be evenly divisible by the number of pages you request

### 3. **Inefficient State Management**

**The Problem:** Reconfiguring NOC state for every operation when you could reuse it.

```cpp
// SLOW - Reconfigures NOC state every iteration
for (uint32_t i = 0; i < num_writes; i++) {
    uint64_t dst_addr = get_noc_addr(x, y, base_addr + i * stride);
    noc_async_write(src_addr, dst_addr, size);
}

// FAST - Set state once, reuse multiple times (for one packet writes)
uint64_t dst_noc_addr = get_noc_addr(x, y, base_addr);
noc_async_write_one_packet_set_state(dst_noc_addr, packet_size);
for (uint32_t i = 0; i < num_writes; i++) {
    noc_async_write_one_packet_with_state(src_addr, base_addr + i * stride);
}

// FAST - For reads with state
noc_async_read_set_state(src_noc_addr);
for (uint32_t i = 0; i < num_reads; i++) {
    noc_async_read_with_state(src_addr + i * stride, dst_addr, size);
}
```

### 4. **Wrong NOC Address Calculation**

**The Problem:** Incorrect address calculations lead to writes to wrong locations or alignment issues.

```cpp
// WRONG - Manual address calculation prone to errors
uint64_t noc_addr = (uint64_t(x) << 32) | (uint64_t(y) << 16) | local_addr;

// CORRECT - Use provided helpers
uint64_t noc_addr = get_noc_addr(x, y, local_addr);
uint64_t dram_addr = get_noc_addr_from_bank_id(bank_id, local_addr);
uint64_t mcast_addr = get_noc_multicast_addr(x_start, y_start, x_end, y_end, local_addr);
```

### 5. **Inefficient Multicast Patterns**

**The Problem:** Using unicast when multicast would be much faster.

```cpp
// SLOW - Multiple unicast writes
for (uint32_t core = 0; core < num_cores; core++) {
    uint64_t dst_addr = get_noc_addr(core_coords[core].x, core_coords[core].y, local_addr);
    noc_async_write(src_addr, dst_addr, size);
}

// FAST - Single multicast write
uint64_t mcast_addr = get_noc_multicast_addr(x_start, y_start, x_end, y_end, local_addr);
noc_async_write_multicast(src_addr, mcast_addr, size, num_cores);
```

### 6. **Understanding Virtual Channels**

Use multiple VCs to separate traffic of different classes, not to parallelize the same-class traffic to the same destination.


**Different bandwidth classes:** Traffic to DRAM vs. Tensix L1 should use different VCs because their bandwidth characteristics differ. However, traffic to Ethernet and Tensix L1 should use the same VC because both have the same bandwidth.

**Flow-control/credit messages:** Should use a VC separate from data. For example, if the traffic indicates how much buffer space was freed, it should be placed on a VC separate from data.

**Available VCs:** Although there are 16 total virtual channels, software can only use a subset of that.
For Unicast operations, valid VC values are 0,1,2,3.
For Multicast operations, VC 4 is the only one which can be used.

Other VCs are reserved for system use.

```cpp
// Example: Separate DRAM (slow) from L1 (fast) traffic
constexpr uint32_t DRAM_VC = 0;        // DRAM has different bandwidth characteristics
constexpr uint32_t L1_VC = 1;          // L1 and Ethernet can share this VC (same bandwidth)
constexpr uint32_t CREDIT_VC = 2;      // Flow control messages

// DRAM operations on separate VC
noc_async_read(dram_addr, l1_addr, size, noc_id, DRAM_VC);

// L1 operations (including Ethernet) can share VC
noc_async_write(l1_src, l1_dst, size, noc_id, L1_VC);

// Credit/flow-control messages separate from data
noc_async_write(credit_msg, dst_addr, sizeof(credit), noc_id, CREDIT_VC);
```

**WHEN MANUAL VC ASSIGNMENT HELPS:**

**1. DRAM vs Compute Traffic Separation**
```cpp
// Prevent slow DRAM reads from blocking fast L1 operations
constexpr uint32_t DRAM_VC = 0;
constexpr uint32_t L1_VC = 2;

noc_async_read(dram_addr, l1_addr, size, noc_id, DRAM_VC);      // Slow
noc_async_write(l1_src, l1_dst, size, noc_id, L1_VC);          // Fast, independent
```

**2. Priority-Based Traffic**
```cpp
// High-priority operations get dedicated VC
constexpr uint32_t HIGH_PRIORITY_VC = 2;
constexpr uint32_t LOW_PRIORITY_VC = 3;

noc_async_read(critical_data_addr, l1_addr, size, noc_id, HIGH_PRIORITY_VC);
noc_async_read(background_addr, buffer_addr, size, noc_id, LOW_PRIORITY_VC);
```

A thing to keep in mind is that software has no way of controlling virtual channel number assignment for response / acknowledgement packets, those will be assigned automatically.

## Performance Optimization Strategies

### 1. **Choose the Right Transfer Size**

**Small transfers (< 8KB WH, < 16KB BH):** Use `noc_async_read_one_packet` / `noc_async_write_one_packet`  
**Large transfers (> 8KB WH, > 16KB BH):** Use regular APIs with proper chunking ( If the data can fit into one packet, the software will issue a noc_async_(read/write)_one_packet, but there will be unnecessary overhead )

### 2. **Virtual Channel Optimization Strategy**

**Use VCs to separate different traffic classes based on bandwidth characteristics:**

```cpp
// Example: Separating traffic by bandwidth class
constexpr uint32_t DRAM_VC = 0;          // DRAM has different bandwidth characteristics
constexpr uint32_t L1_ETHERNET_VC = 1;   // L1 and Ethernet share same bandwidth class
constexpr uint32_t CREDIT_VC = 2;        // Flow control separate from data

void optimized_data_movement() {
    // DRAM operations on separate VC (different bandwidth class)
    noc_async_read(dram_addr, l1_addr, size, noc_id, DRAM_VC);
    
    // L1 operations (same bandwidth class as Ethernet)
    noc_async_write(l1_src, l1_dst, size, noc_id, L1_ETHERNET_VC);
    
    // Flow control messages separate from data
    noc_async_write(credit_msg, dst_addr, sizeof(credit), noc_id, CREDIT_VC);
}
```

## API Quick Reference

### Core Read APIs
- `noc_async_read()` - General purpose async read
- `noc_async_read_barrier()` - Wait for read completion
- `noc_async_read_set_state()` / `noc_async_read_with_state()` - Stateful reads

### Core Write APIs  
- `noc_async_write()` - General purpose async write
- `noc_async_write_multicast()` - Broadcast to multiple cores
- `noc_async_write_barrier()` - Wait for write completion
- `noc_async_writes_flushed()` - Ensure writes are sent (faster than barrier)

### Circular Buffer APIs
- `cb_reserve_back()` / `cb_push_back()` - Producer side
- `cb_wait_front()` / `cb_pop_front()` - Consumer side  
- `get_write_ptr()` / `get_read_ptr()` - Get memory addresses

### Address Generation
- `get_noc_addr()` - Convert coordinates to NOC address
- `get_noc_addr_from_bank_id()` - DRAM bank addressing
- `get_noc_multicast_addr()` - Multicast address generation
