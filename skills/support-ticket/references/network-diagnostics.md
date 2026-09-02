# Network Support Diagnostics

Use this reference for network, routing, latency, loss, throughput, transport, tunnel, or distributed-consensus symptoms. Run only the smallest relevant part of the matrix and only against systems or provider-supplied targets in scope.

## Frame the Failure

Record source, destination, direction, address family, protocol, ports, flow identity, packet size, offered rate, concurrency, time window, base RTT, and application impact. Distinguish these commonly conflated questions:

- reachability versus usable application traffic;
- ingress versus egress;
- latency versus queueing versus loss versus short forwarding outages;
- aggregate capacity versus per-flow behavior;
- bits per second versus packets per second;
- random loss versus flow-hashed selection versus time-correlated loss;
- host, access network, transit, peering, and return-path behavior.

Do not assume the forward and return paths use the same networks. A traceroute observes the path taken by its probe type and the return path taken by each reply.

## Build the Smallest Comparison Matrix

Select only comparisons that separate live hypotheses:

| Question | A | B | Hold constant |
| --- | --- | --- | --- |
| Scope | affected destination | known-good destination | source, protocol, window |
| Direction | forward test | reverse test | endpoints, tool, load |
| Protocol | ICMP | TCP or UDP | endpoints, window |
| Flow hash | pinned five-tuple | varied ports | time and application |
| Capacity | one stream | parallel streams | endpoints and duration |
| Rate unit | same bit rate, different packet rate | same packet rate, different bit rate | direction and path |
| Time structure | endpoint counters | dual-ended packet sequence/timestamps | exact UTC window |

A control in another prefix, site, host class, or transit can localize policy or routing scope. It does not by itself identify the internal mechanism.

## Use Tools for What They Can Prove

- `ping` establishes ICMP echo behavior only. Healthy ICMP does not clear TCP, UDP, tunnels, application traffic, QoS, ECMP, or protocol-specific paths.
- `mtr` and traceroute localize where reply behavior changes. Loss or latency at an intermediate hop that does not persist to later hops is usually reply de-prioritization, not end-to-end forwarding loss.
- TCP-mode `mtr`, repeated connects, or application requests can expose behavior hidden from ICMP. Record distributions, not only averages.
- `iperf3` can compare direction with `-R`, concurrency with `-P`, and paced UDP with `-u -b`. Single-stream scaling across parallel streams argues against one aggregate cap but may indicate per-flow loss, policy, or hashing.
- `ss -tin`, interface counters, `tc -s`, softnet counters, socket error counters, and hypervisor or switch counters help identify where loss is counted. Sample before and after the exact test window.
- `tracepath` and path traces can expose different transit choices, but unchanged visible hops after recovery do not prove the internal route or device was unchanged.
- Dual-ended packet captures can distinguish sender omission, path loss, receiver delivery, receiver-socket drops, reordering, and contiguous blackout runs. Capture headers or a short snap length unless payload is required and authorized.

Keep exact commands and raw output with the evidence. Interpret tool output in the context of the probe protocol and return path.

## Test Flow Selection

A bimodal distribution suggests two discrete outcomes and deserves a hashing or policy test. Pin the same source port and five-tuple across spaced rounds, then compare with deliberately varied ports.

- A stable fast or slow outcome for the same tuple supports deterministic flow selection.
- Outcomes that move with time regardless of tuple support a time-varying condition instead.
- One fixed UDP tunnel tuple can remain pinned to a bad path even when repeated TCP connects occasionally select a good one.

Do not infer where the hash occurs until direction is established. On return traffic, a client source port may become a destination port and still participate in an upstream egress hash.

## Separate Rate, Burst, and Loss Mechanisms

Do not vary packet size, packet rate, bit rate, burst size, pacing, and concurrency together and then attribute the result to one of them.

- To test a packet-rate ceiling, compare different packet rates at the same bit rate and different bit rates at the same packet rate.
- To test burst sensitivity, hold average rate constant and vary burst structure; verify what reaches the wire when offloads can coalesce traffic.
- To test an aggregate shaper, compare one stream with parallel streams and measure aggregate throughput.
- To test congestion, look for load-correlated delay, loss, and queue counters. A throughput change alone is not a mechanism.
- To distinguish random loss from short outages, compare sequence numbers and timestamps. Contiguous runs and periodic gaps can be more actionable than an average loss percentage.

Treat sender pacing or congestion-control changes as experiments or mitigations unless they repair the provider fault. If changing two mechanisms together improves throughput, isolate them before drawing a causal conclusion.

## Correlate the Provider Boundary

Once endpoint and host counters cannot account for measured loss, name the remaining segment and ask the provider for the counter or event that can. Useful requests include:

- per-interface and per-member errors, discards, queue drops, and state changes;
- qdisc, policer, QoS, rate, burst, and packet-rate configuration;
- tap, vhost, bridge, vswitch, softnet, conntrack, IRQ, and softirq counters;
- route, next-hop, ECMP, LAG, BFD, spanning-tree, HA, and reconvergence events;
- exact configuration or routing differences between an affected and clean prefix or host.

Offer one coordinated test with an exact UTC window and a signature the counter should reproduce. If the provider performs a change, rerun the same failing profile first, then controls and the real application. Test every affected scope before accepting closure.
