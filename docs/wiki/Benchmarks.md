# Benchmarks

Every number here is produced by the `benchmark` workflow in the repository, on a GitHub-hosted runner, comparing the latest zeptun release with sing-box 1.15 on its own TCP/IP stack, over the same SOCKS5 server.

## Machine

| property | value |
|---|---|
| runner image | ubuntu24 20261101.360.1 |
| cpu | AMD EPYC 9V45 96-Core Processor |
| cpu cores | 4 |
| memory | 15.6 GB |
| kernel | Linux 7.0.0-1017-azure |
| zig | 0.17.0 |
| date | 2026-11-04 |
| zeptun | v1.1.3 (Noisemux/zeptun release) |
| sing-box | v1.15.0-alpha.13 (sing-tun stack) |

## Method

zeptun is the latest release binary of Noisemux/zeptun, and sing-box is the latest 1.15.0 alpha with no tun `stack` option, so it runs on sing-tun's own TCP/IP stack. Two network namespaces joined by a veth pair. Traffic enters the tunnel device (MTU 8500), leaves through the same SOCKS5 server (hev-socks5-server) for both engines, and reaches the servers in the second namespace. Engines are interleaved: each round starts every engine once and runs every scenario against it, so background noise spreads evenly. CPU comes from `/proc/<pid>/stat` and memory from `VmRSS`, summed over all processes of an engine.

Duration 8 s per scenario, 2 rounds, 4 queues. Memory is read twice: the highest sample while the scenario runs, and again after 5 s of idle, which shows whether an engine gives the memory back.

## Throughput

![throughput](res/throughput.svg)

## CPU

![cpu](res/cpu.svg)

## Request and response

![transactions](res/transactions.svg)

![latency](res/latency.svg)

## UDP

![udp](res/udp.svg)

## Memory

Memory is read twice per scenario: the highest sample while the load runs, and
again after a few seconds of idle. The second reading is what shows whether an
engine hands the memory back or keeps it for the life of the process.

![memory](res/memory.svg)

## Scenarios

| Scenario | What it measures |
|---|---|
| `tcp-up-1`, `tcp-up-10` | bulk upload over one and ten streams |
| `tcp-down-1`, `tcp-down-10` | the same downstream |
| `rr` | request and response on a held connection, one at a time |
| `rr-8x1k` | eight connections exchanging 1 KB messages |
| `crr` | connect, exchange, close, repeatedly |
| `udp-100k` | 100k datagrams per second, echoed |
| `udp-gso-100k` | the same with segmentation offload on the client |

## Raw results

| engine | scenario | median | cpu % | max rss MB | rss after idle MB |
|---|---|---:|---:|---:|---:|
| zeptun-userspace | tcp-up-1 | 29.520 Gbit/s | 74 | 2.9 | 1.4 |
| zeptun-userspace | tcp-up-10 | 34.370 Gbit/s | 141 | 9.7 | 2.4 |
| zeptun-userspace | tcp-down-1 | 19.663 Gbit/s | 99 | 2.6 | 2.5 |
| zeptun-userspace | tcp-down-10 | 33.312 Gbit/s | 185 | 6.4 | 2.6 |
| zeptun-userspace | rr | 15547 tps p50=63us p99=79us p99.9=260us  | 34 | 2.6 | 2.6 |
| zeptun-userspace | rr-8x1k | 57658 tps p50=130us p99=304us p99.9=432us  | 95 | 2.7 | 2.6 |
| zeptun-userspace | crr | 3419 tps p50=280us p99=340us p99.9=608us  | 45 | 4.5 | 4.3 |
| zeptun-userspace | udp-100k | 99842 echo pps (99.8% of 100000 sent)  | 41 | 4.9 | 4.2 |
| zeptun-userspace | udp-gso-100k | 99973 echo pps (100.0% of 99988 sent)  | 25 | 5.1 | 4.2 |
| zeptun-hybrid | tcp-up-1 | 18.105 Gbit/s | 99 | 2.0 | 1.8 |
| zeptun-hybrid | tcp-up-10 | 30.540 Gbit/s | 189 | 3.5 | 2.6 |
| zeptun-hybrid | tcp-down-1 | 18.748 Gbit/s | 99 | 2.9 | 2.4 |
| zeptun-hybrid | tcp-down-10 | 33.514 Gbit/s | 226 | 3.6 | 2.7 |
| zeptun-hybrid | rr | 13623 tps p50=71us p99=88us p99.9=364us  | 42 | 2.7 | 2.5 |
| zeptun-hybrid | rr-8x1k | 50726 tps p50=136us p99=380us p99.9=496us  | 144 | 2.6 | 2.2 |
| zeptun-hybrid | crr | 2653 tps p50=356us p99=432us p99.9=680us errors=182  | 68 | 6.7 | 6.4 |
| zeptun-hybrid | udp-100k | 99984 echo pps (100.0% of 99992 sent)  | 58 | 7.3 | 6.6 |
| zeptun-hybrid | udp-gso-100k | 99974 echo pps (100.0% of 99988 sent)  | 25 | 7.3 | 6.8 |
| singbox-native | tcp-up-1 | 30.491 Gbit/s | 94 | 61.8 | 61.8 |
| singbox-native | tcp-up-10 | 25.508 Gbit/s | 96 | 65.6 | 65.6 |
| singbox-native | tcp-down-1 | 17.958 Gbit/s | 100 | 79.1 | 69.3 |
| singbox-native | tcp-down-10 | 16.924 Gbit/s | 100 | 103.4 | 103.4 |
| singbox-native | rr | 14714 tps p50=66us p99=84us p99.9=356us  | 40 | 103.4 | 103.4 |
| singbox-native | rr-8x1k | 47139 tps p50=146us p99=544us p99.9=1520us  | 85 | 105.1 | 105.1 |
| singbox-native | crr | 2536 tps p50=376us p99=624us p99.9=1568us  | 81 | 182.9 | 182.9 |
| singbox-native | udp-100k | 0 echo pps (0.0% of 99984 sent)  | 54 | 187.2 | 74.2 |
| singbox-native | udp-gso-100k | 0 echo pps (0.0% of 99992 sent)  | 27 | 74.3 | 72.8 |

zeptun: startup 2 ms, idle 840 KB, 5 wakeups in 20 s | tcp 1000: 2792 KB (conns: 1000/1000 established, 0 failed, 3343 conn/s) | udp 1000: 2428 KB (udp flows: 1000/1000 answered, 7861 flows/s)
singbox-native: startup 38 ms, idle 53912 KB, 73 wakeups in 20 s | tcp 1000: 64903 KB (conns: 1000/1000 established, 0 failed, 2467 conn/s) | udp 1000: 68087 KB (udp flows: 0/1000 answered, 0 flows/s)

## Reproducing

```sh
gh workflow run benchmark.yml --repo hiddify/zeptun -f duration=8 -f repeat=2
```
