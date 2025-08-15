Virtual Channels
=================

## Summary

This document evaluates whether alternating across multiple NoC virtual channels (VCs) improves throughput for unicast writes and read requests on Wormhole B0 and Blackhole. Across one-to-one, one-to-all, and all-to-all traffic patterns, we observed no performance benefit from using multiple VCs versus a single VC. In some cases, performance degraded as the number of VCs increased. Architectural details of the router allocation and input buffer sharing explain these results.

## Background

In Wormhole and Blackhole, there are two physical NoC (Network-on-Chip) channels, each comprising 16 virtual channels. Of these 16 VCs, 8 are non-dateline and 8 are dateline. Each non-dateline VC has a corresponding dateline VC to avoid deadlock.

Among the 8 non-dateline VCs, 4 are designated for unicast traffic (i.e., unicast write packets of 257 flits and read-request flits). A commonly proposed optimization is to alternate traffic across multiple VCs to alleviate head-of-line blocking and improve throughput, especially for small transfers between Tensix cores.

## Methodology

We performed sweep tests in the following nested order (number of transactions per test = 256 unless otherwise noted):
- NoC ID: 0, 1
  - Transaction size: 1 page, 2 pages, …, up to 4096 pages (page size is 32 B for Wormhole, 64 B for Blackhole)
    - Number of alternated VCs: 1, 2, 3, 4

Traffic patterns were evaluated for both unicast writes and read requests:
- One-to-one (one source to one destination)
- One-to-all (one source to all destinations)
- All-to-all (all sources to all destinations)

Notation: 'to' denotes unicast write tests; 'from' denotes read request tests.

Note on read responses: Read response VCs are selected by hardware; software cannot control the response VC. Consequently, VC sweeps are not expected to materially affect read-request results.

## Results

### Unicast write tests

#### Wormhole B0

| Test type          | Plot |
|--------------------|------|
| One-to-one         | <img src="plots/wormhole_b0/One to One Virtual Channels.png" alt="WH One-to-One Virtual Channels" width="1200"> |
| One-to-all unicast | <img src="plots/wormhole_b0/One to All Unicast Virtual Channels.png" alt="WH One-to-All Virtual Channels" width="1200"> |
| All-to-all         | <img src="plots/wormhole_b0/All to All Virtual Channels.png" alt="WH All-to-All Virtual Channels" width="1200"> |

#### Blackhole

| Test type          | Plot |
|--------------------|------|
| One-to-one         | <img src="plots/blackhole/One to One Virtual Channels.png" alt="BH One-to-One Virtual Channels" width="1200"> |
| One-to-all unicast | <img src="plots/blackhole/One to All Unicast Virtual Channels.png" alt="BH One-to-All Virtual Channels" width="1200"> |
| All-to-all         | <img src="plots/blackhole/All to All Virtual Channels.png" alt="BH All-to-All Virtual Channels" width="1200"> |

- Unicast writes — one-to-one: No material difference across VC counts; no congestion → VCs provide no benefit, as expected.
- Unicast writes — one-to-all: Largely similar behavior to one-to-one; this kernel serializes destinations, so VC alternation does not help.
- Unicast writes — all-to-all: Bandwidth varies irregularly with size and VC count; see Discussion for causes.

### Read request tests

Note: Read response VCs are hardware-selected; minimal sensitivity to VC count is expected.

#### Wormhole B0

| Test type     | Plot |
|---------------|------|
| One-from-one  | <img src="plots/wormhole_b0/One from One Virtual Channels.png" alt="WH One-from-One Virtual Channels" width="1200"> |
| One-from-all  | <img src="plots/wormhole_b0/One from All Virtual Channels.png" alt="WH One-from-All Virtual Channels" width="1200"> |
| All-from-all  | <img src="plots/wormhole_b0/All from All Virtual Channels.png" alt="WH All-from-All Virtual Channels" width="1200"> |

#### Blackhole

| Test type     | Plot |
|---------------|------|
| One-from-one  | <img src="plots/blackhole/One from One Virtual Channels.png" alt="BH One-from-One Virtual Channels" width="1200"> |
| One-from-all  | <img src="plots/blackhole/One from All Virtual Channels.png" alt="BH One-from-All Virtual Channels" width="1200"> |
| All-from-all  | <img src="plots/blackhole/All from All Virtual Channels.png" alt="BH All-from-All Virtual Channels" width="1200"> |

- Read requests — one-from-one and one-from-all: No meaningful throughput gain from additional VCs.
- Read requests — all-from-all: Erratic bandwidth due to response ordering effects and VC selection for responses; see Discussion.

## Discussion and interpretation

Contrary to the initial expectation that multiple VCs would increase throughput by reducing head-of-line blocking, the observed results show no benefit, and sometimes degradation, when alternating among multiple VCs within the same traffic class. This is consistent with prior Quasar results and can be explained by two implementation details of the NoC:

1. Router VC allocation ordering
   - To meet timing, the router arbitrates among VCs before arbitrating among input and output ports. If a blocked VC is selected during VC arbitration, the router may incur cycles where no flit advances, even if another VC could have routed. This introduces idle cycles that negate the intended benefit of using multiple VCs.

2. Input buffer sharing across VCs
   - Input buffers are shared among VCs and sized such that three active VCs can sustain 100% bandwidth. If three VCs become stalled, all other VCs will drop to 1/16th bandwidth. With 1 VC, the hardware effectively uses 2 VCs (the non-dateline VC and its paired dateline VC). With 2 VCs, we effectively engage 4 VCs, which can trigger buffer pressure and expose the shared-buffer limits, degrading performance. As more VCs are alternated (e.g., 3 or 4), we observe even lower bandwidth due to exacerbated shared-buffer pressure.

Additional notes grounded in the results:
- One-to-one: With no contention, VC choice does not matter; performance is identical across VC counts.
- One-to-all: The evaluated kernel sends to one destination at a time (serialized across destinations), so VC alternation provides little to no benefit.
- All-from-all (read responses): The erratic nature stems from perturbations in response ordering and the somewhat random selection of response VCs, which can create transient congestion patterns.

## Recommendations

- Use multiple VCs to separate traffic of different classes, not to parallelize the same-class traffic to the same destination.
  - Different bandwidth classes: e.g., traffic to DRAM vs. Tensix L1 should use different VCs because their bandwidth characteristics differ. However, traffic to Ethernet and Tensix L1 should use the same VC because both have the same bandwidth.
  - Flow-control/credit messages should use a VC separate from data. For example, if the traffic indicates how much buffer space was freed, it should be placed on a VC separate from data. Historically, VC 2 was used for this in Buda.
- Do not split otherwise identical debug and production data into different VCs if they target the same destination; they are effectively the same traffic class.
- For kernels that currently serialize destinations, consider algorithmic changes (e.g., randomized destination order) to reduce serialization before attempting VC alternation.

## Open questions and next steps

- Bandwidth drop at small transaction counts: When the number of transactions is 4, we observed a notable bandwidth drop beyond 4096 B/transaction on Wormhole and 2048 B/transaction on Blackhole (NoC 0). The most likely cause is unintended serialization. To mitigate, randomize destination and VC selection to break serialization.

Example approach to randomization per iteration:

```c++
uint32_t current_virtual_channel = (i + 9 * my_y + my_x) % num_virtual_channels;
uint32_t sub_idx = (j + 9 * my_y + my_x) % num_subordinates;
subordinate_x_coord = get_arg_val<uint32_t>(sub_idx * 2);
subordinate_y_coord = get_arg_val<uint32_t>(sub_idx * 2 + 1);
```

- Follow-up experiments:
  - With the number of transactions set to 4, re-run one-to-all and all-to-all with randomized destination/VC selection to quantify the impact of reduced serialization.
  - Evaluate a configuration where flow-control messages are placed on a dedicated VC, separate from data, to validate the recommendation above. This may be intertwined with the atomic APIs investigation.
  - Probe sensitivity to buffer occupancy by throttling injection rates to determine the onset of shared-buffer limits.
  - Add a VC sweep for loopback traffic for completeness (low expected value; implicitly exercised by one-to-one and one-to-all unicast)
  - After executing these steps, update this document with revised results, analysis, and recommendations.
  