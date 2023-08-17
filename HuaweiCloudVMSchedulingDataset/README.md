# Overiew

This folder comprises the public release of Huawei Cloud traces collected between 2022 and 2023. These traces have been created to support research in various areas, such as Quality of Service (QoS)-aware online and offline scheduling of Virtual Machines (VM), as well as the management of cloud computing value chains. The dataset includes four distinct classes of traces, each tailored to address a specific real-life problem commonly encountered by cloud providers.

+ Hotspot Resolution: This trace includes hotspot Physical Machine (PM) data collected from Huawei Cloud in 2022. It also provides information about the VMs running on the PMs, including their utilization over a specific time period. The trace is designed to facilitate offline scheduling and migration decisions for VMs, aiming to mitigate PM hotspots.

+ VM Online Scheduling: This trace consists of comprehensive VM information from Huawei Cloud in 2022, with over 100,000+ VM entries available. It includes 297 testing instances, where each instance represents a specific scenario involving a queue of VMs that need to be allocated across a fixed number of PMs.

+ QoS Modeling: This trace focuses on QoS (Quality of Service) modeling and evaluation. It provides Steal Time (ST) data, real utilization data of VMs, and the mapping of VMs to PMs. ST represents the percentage of time a VM waits for service when it is ready to be served—an important metric for assessing QoS performance.

+ VM Offline Scheduling: This trace contains a total of 117 instances, each consisting of a variable number of PMs (ranging from 10 to 200). Additionally, it includes information about the VMs running on these PMs, along with their real vCPU utilization over a specified time horizon. The trace aims to support VM offline scheduling tasks, such as VM reallocation and migration among PMs. These operations can be performed to achieve specific objectives, such as balancing hotspot risks or emptying more PMs.

## Traces Description

### Hotspot Resolution

A hotspot in our dataset is defined as a timestamp when the cumulative utilized capacity (vCPU usage) of VMs on a PM exceeds a predetermined safety threshold of the PM's total capacity. The trace includes information about a total of 1,295 PMs, as well as the VMs running on these PMs. Our primary objective is to release an optimal set of VMs on PMs that are currently experiencing or likely to experience hotspots while minimizing the migration cost.

The trace is provided in JSON format, with each file representing a PM. Each line within the file represents a VM running on the respective PM and includes the following fields:

+ **host_vcpus**: the total number of CPU cores available on its hosted PM.
+ **time_start**: the start time of recording the utilization of a VM.
+ **time_end**:  the end time of recording the utilization of a VM.
+ **vm_util**: the real utilized capacity of a VM at each timestamp. For instance, if a VM has a flavor with 2 CPU cores and its utilization at a specific timestamp is 25%, the vm_util value would be calculated as 2 * 0.25 = 0.5. In cloud practice, the vm_util is collected and aggregated every 5 minutes, resulting in a total of 288 timestamps in a day.

### VM Online Scheduling

In cloud resource scheduling and management, efficiency and quality are two vital yet conflicting objectives. Cloud providers, such as Huawei, prioritize quality by avoiding hotspots and aim to optimize and increase the utilization efficiency of PM resources without compromising quality. The typical online VM scheduling problem in cloud practice can be stated as follows: given a fixed number of PMs and a queue of arriving VMs, the goal is to place as many VMs as possible onto the PMs while ensuring that no hotspots occur.

The trace consists of a total of 297 instances, where each instance is represented by a JSON file named in the format X-1.json, X-2.json, and X-3.json. Here, X represents the number of PMs capable of hosting the arriving VMs, ranging from 2 to 100. For example, 50-1.json indicates that there are 50 available PMs to host the arriving VMs. For each value of X, there are three replicas denoted by the suffixes 1, 2, and 3.

There are multiple flavors of PM available in our cloud service provider. Readers can refer to our online paper, which we will provide the link for below, to learn about the specific flavor we used for academic and testing purposes. However, they are also free to use other typical flavors as per their requirements.

Each JSON file contains information about the VMs waiting to be assigned to PMs, and the fields for these VMs are as follows (each line represents a VM):

+ **Created_at_point**: the timestamp when the VM arrives. The list is already sorted in ascending order based on the arrival times.
+ **memory**: the memory capacity of the VM defined by its flavor, measured in gigabytes (GB).
+ **duration_point**: the timestamps indicating the duration of the VM's usage. Each timestamp represents a 5-minute interval in practice.
+ **vm_util**: the real utilized capacity of the VM at each timestamp, with a similar meaning as the Hotspot Resolution trace. The length of the list is equal to the value of "duration_point".

### QoS Modelling

The Steal Time (ST) metric is an efficient measure for evaluating QoS performance, particularly in relation to the CPU dimension. ST refers to the amount of time a virtual machine (VM) is ready to receive CPU resources but has to wait. It represents the CPU time that is taken away from the VM by the hypervisor to service other VMs running on the same physical machine (PM). If critical processes are delayed, a high ST can lead to performance degradation for the VM.

One key distinction of our trace is that we study ST at the NUMA-Node level. Some VMs are deployed on a single NUMA-Node, while others, especially larger VMs, may be deployed across two NUMA-Nodes of a PM.

The trace includes the following fields:

+ **instance_id**: UUID of the VM
+ **flavor_group**: the VM type, categorized as F1 and F2. F1 represents the "dedicated" flavor, where a single VM does not share CPU cores with other VMs. F2 represents the "shared" flavor, where CPU resources are shared among multiple VMs. In cloud computing, dedicated flavor VMs receive higher priority, and their resources are exclusively allocated from the PMs. The remaining resources are then utilized to fulfill the requirements of shared instances.
+ **vcpus**:the number of vCPU cores defined by the VM's flavor.
+ **pod_host**: the UUID of the PM on which the VM is hosted.
+ **times**: a total of 289 timestamps. A value of 0 indicates that the data for steal time and CPU utilization is missing, while a value of 1 indicates that the data for that timestamp has been successfully collected.
+ **cpu_util**: the utilization (not the utilized capacity) of the VM at each timestamp. For example, a value of 25 means that at a specific timestamp, a VM with a flavor of 4 cores has a real utilization of 25%, resulting in a utilized capacity of 4 * 0.25 = 1.
+ **steal_time**: the steal time, expressed as a percentage, for the VM at each timestamp. For example, a value of 10 means that the steal time at a specific timestamp is 10%. The first value of 0 in the steal time list denotes a missing point (corresponding to the first number in the "times" field being 0), while the subsequent 0 values indicate that the actual steal time at those timestamps is 0.
+ **numa_typology**: information about the NUMA nodes hosting the VM. The "cell_id" field indicates the ID of the NUMA node. If the VM is hosted by a single NUMA node, it utilizes all of its CPU capacity from that node. If the VM is hosted by two NUMA nodes, it splits its utilized CPU capacity evenly between the two nodes.

### VM Offline Scheduling

The trace consists of 117 instances, each named in the format X-1.json, X-2.json, and X-3.json. In this naming convention, X represents the number of PMs in that particular instance, and the suffixes 1, 2, and 3 denote different replications. Each instance consists of multiple PMs, each hosting a certain number of VMs. The trace provides historical utilization data for each VM. The trace can be used for VM repacking through live-migration to achieve various goals, including hotspot risk balancing, energy-saving by freeing up more PMs, or densely packing existing VMs to make room for larger upcoming VMs.

Each instance contains PMs with the following fields:

+ **pm_id**: the UUID of the PM.
+ **threshold**: the hotspot threshold of the PM, expressed in a unit of 1. For example, if the value is 0.7, it means that if the cumulative utilized capacity of VMs exceeds 70% of the total CPU capacity of the PM, it is considered a hotspot.
+ **vcpus**: the total number of CPU cores of the PM.
+ **memory**: the total amount of memory in GB available on the PM.
+ **cpu_usage**:  the real utilization of each VM at each timestamp, expressed as a percentage. For example, if the value is 25, it means that at a specific timestamp, the CPU utilization of that particular VM is 25%. There are a total of 300 timestamps provided for each VM in the trace.

### Quick Links by paper

+ Trace "Hotspot Resolution" for the paper **"Hotspot Resolution in Cloud Computing: A $\Gamma$ Robust Knapsack Approach for Virtual Machine Migration"** (forthcoming)

+ Trace "VM Online Scheduling" for the paper **"Optimize resource utilization in Data Centers: A Robust Bin Packing-based approach for Online Virtual Machine Provisioning"** (forthcoming)

+ Trace "VM Online Scheduling" for the paper **"Online Virtual Machine Provisioning under Uncertainty: A Robust Approach to Ultimate Resource Utilization"** (forthcoming)

## Contributors

The dataset is a result of collaboration between the Computing and Networking Innovation Lab and the Elastic Compute Service Domain Program at Huawei Cloud.

The research and innovative work involved a combined effort from the Computing and Networking Innovation Lab at Huawei Cloud and the Mathematical Modeling and Optimization Algorithm Laboratory at Huawei 2012. The main researchers involved in this project are Jiaxi Wu, Wenquan Yang, Andrey Tikhonov, Gudkov Andrei, Romanov Stepan, and Ponomareva Elizaveta.

We would like to express our gratitude to Lei Zhu and Xueqi Wu from the Elastic Compute Service Domain Program for his valuable support in data collection and publication. We also extend our thanks to Yongqiang Yang, Shuming Jing, Xuqi Zhu, and Zhen Fang from the Computing and Networking Innovation Lab for their collaboration and contributions to this work.

Contact us
If you have any questions or would like to reach out to us, please feel free to send us an email. We welcome any suggestions or opportunities for collaboration:

+ <wujiaxi1@huawei.com>
+ <yangwenquan3@huawei.com>
