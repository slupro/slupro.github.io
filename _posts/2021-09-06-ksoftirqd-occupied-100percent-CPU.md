---
layout: post
title:  "ksoftirqd occupied 100% CPU"
subtitle:   "ksoftirqd 占用 100%CPU，造成系统瓶颈"
date:   2021-09-06 13:00:00 +10:00
author: "Steven Lu @slupro"
categories: [性能调优]  # up to 2.
tags: [ksoftirqd, performance]  # TAG names should always be lowercase, 0 to infinity.
---

### Issue

In a performance testing, the Linux system is not in high loads (40% of total CPUs is in use), but our service can't receive more requests and the ssh connections frequently disconnected.

![](/assets/img/ksoftirqd-occupied-100percent-CPU/2021-09-06-18-22-04.png)

### Investigation

The testing environment was built on a KVM, and it was assigned 24 CPU cores. In ```top``` output, ksoftirqd/17 occupied 98% CPU. ksoftirqd is used to handle the interrupt from hardware, so maybe this process is the bottleneck of the system.

![](/assets/img/ksoftirqd-occupied-100percent-CPU/2021-09-06-18-56-30.png)

There are 24 ksoftirqd processes related to 24 CPU cores, but only ksoftirqd/17 is very busy. So let's check what ksoftirqd/17 is working for. ```/proc/interrupts``` recorded the interrupt information. Our NIC is ens3. We can see int 11 is working for ens3, and a large number of int 11 occured in CPU17, which is related with ksoftirqd/17.

![](/assets/img/ksoftirqd-occupied-100percent-CPU/2021-09-06-21-57-40.png)

Obviously the NIC interrupts are not distributed to multiple CPUs. All NIC interrupts are only sent to ksoftirqd/17, and cause CPU 17 is too busy to process more packets. 

The other CPUs are not fully utilized, so we need to distribute the NIC interrupts to other CPUs to improve the system performance.

### Solve it

Run ```ethtool -l ens3``` to query NIC channels:

```
# ethtool -l ens3
Channel parameters for ens3:
Cannot get device channel parameters
: Operation not supported
```

It looks this KVM was not set to support multiple channels on NIC, or the physical NIC can't support multiple channels. Irqbalance service is running, but it seems irqbalance can't handle this case. We need to distribute the NIC interrupts manually by using Receive Packet Steering(RPS) and Receive Flow Steering(RFS).

RPS works in Linux kernel to distribute NIC interrupts to multiple CPUs. RFS is to increase datacache hitrate by steering kernel processing of packets to the CPU where the application thread consuming the packet is running. The full document about network scaling could found at https://www.kernel.org/doc/Documentation/networking/scaling.txt.

Just list the steps I did:

1. Disable irqbalance service.

```
# systemctl stop irqbalance
# systemctl disable irqbalance
```

2. Set SMP IRQ affinity. We could assign which CPUs can process the specific interrupt. In this VM, int 11 works for NIC interrupts, and I want all CPU cores could process the NIC interrupts.

```
# echo "0-23" > /proc/irq/11/smp_affinity_list
```

* Alternatively, you could set the smp_affinity by hex value. We can check the last command by ```cat /proc/irq/11/smp_affinity```, and get "ffffff".

3. Configure RPS. "ffffff" is a bitmap of CPUs.

```
# echo "ffffff" >  /sys/class/net/ens3/queues/rx-0/rps_cpus
```

4. Configure RFS. For a single queue device, we set rps_sock_flow_entries and rps_flow_cnt value with the same value for a good performance.

```
# sysctl -w net.core.rps_sock_flow_entries=32768
# echo 32768 > /sys/class/net/ens3/queues/rx-0/rps_flow_cnt
```

### All done

The NIC interrupts are distributed to multiple CPU cores, so our backend service can handle more requests and the whole system is fully utilized.

![](/assets/img/ksoftirqd-occupied-100percent-CPU/2021-09-06-23-41-27.png)
