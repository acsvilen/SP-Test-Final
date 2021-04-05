
StorPool User Guide 19
======================

Document version 2019-08-05

1. StorPool Overview
--------------------

StorPool **is distributed block storage software**. It pools the attached storage (HDDs, SSDs or NVMe drives) of standard servers to create a single pool of shared storage. The StorPool software is installed on each server in the cluster. **It combines the performance and capacity of all drives attached to the servers into one global namespace**.

`StorPool's version 19 has been released in November 2019. <https://storpool.com/news/storpools-new-release-19>`_ The new version has numerous improvements and makes the leading block storage SDS even better. 

Notable features of the new version are:

* Large scale deployments;
* Added Windows CSV support;
* Lower latency for NVMe storage; and
* Improved Kubernetes bare-metal support.

StorPool **provides standard block devices**. You can create one or more volumes through its sophisticated volume manager. StorPool is compatible with ``ext3`` and ``XFS`` file systems and with *any* system designed to work with a block device, e.g. databases and cluster file systems (like OCFS and GFS). 

StorPool **can also be used with no file system**, for example when using volumes to store VM images directly or as LVM physical volumes.

**Redundancy** is provided by multiple copies (replicas) of the data written synchronously across the cluster. Users may set the number of replication copies. We recommend **3 copies as a standard** and 2 copies for data that is less critical. The replication level directly correlates with the number of servers that may be down without interruption in the service. For replication 3 the number of the servers (see `Fault sets <https://kb.storpool.com/user_guides/19.01/user_guide_19.01.html#fault-sets>`_) that may be down simultaneously without losing access to the data is 2.

StorPool **protects data and guarantees its integrity by a 64-bit checksum and version for each sector on a StorPool volume or snapshot**. StorPool provides a very high degree of flexibility in volume management. Unlike other storage technologies, such as RAID or ZFS, StorPool does not rely on device mirroring (pairing drives for redundancy). So every disk that is added to a StorPool cluster adds capacity and improves the performance of the cluster, not just for new data but also for existing data. Provided there are sufficient copies of the data, drives can be added or taken away with no impact to the storage service. Unlike rigid systems such as RAID, StorPool does not impose any strict hierarchical storage structure dictated by the underlying disks. StorPool simply creates a single pool of storage that utilises the full capacity and performance of a set of commodity drives.

2. Architecture
---------------

StorPool **works on a cluster of servers in a distributed shared-nothing architecture**. All functions are performed by all servers on an equal peer basis. It works on standard off-the-shelf servers running GNU/Linux.

**Each storage node is responsible for data stored on its local drives**. Storage nodes collaborate to provide the storage service. StorPool provides a shared storage pool combining all the available storage capacity. It uses synchronous replication across servers. The StorPool client communicates in parallel with all StorPool servers. The StorPool iSCSI target provides access to volumes exported through it to other initiators.

The software consists of two parts – **a storage server and a storage client** – that are installed on each physical server (host, node). The storage client might be the native block on Linux based systems or the iSCSI target for other systems. Each host can be a storage server, a storage client, iSCSI target, or any combination. To storage clients the StorPool volumes appear as block devices under the ``/dev/storpool/`` directory and behave as normal disk devices. The data on the volumes can be read and written by all clients simultaneously; its consistency is guaranteed through a synchronous replication protocol. Volumes may be used by clients as they would use a local hard drive or disk array.


3. Feature Highlights
---------------------

3.1. Scale-out, not Scale-Up
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The StorPool solution is fundamentally about *scaling out* (by adding more drives or nodes) rather than scaling up (adding capacity by replacing a storage box with larger storage box). This means StorPool can scale independently by IOPS, storage space and bandwidth. There is no bottleneck or single point of failure. StorPool can grow without interruption and in small steps – a drive, a server and/or a network interface at a time.

3.2. High Performance
^^^^^^^^^^^^^^^^^^^^^

StorPool **combines the IOPS performance of all drives** in the cluster and **optimizes drive access patterns** to provide low latency and handling of storage traffic bursts. The load is distributed equally between all servers through striping and sharding.

3.3. High Availability and Reliability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

StorPool **uses a replication mechanism that slices and stores copies of the data on different servers**. For primary, high performance storage this solution has many advantages compared to RAID systems and provides considerably higher levels of reliability and availability. In case of a drive, server, or other component failure, StorPool uses some of the available copies of the data located on other nodes in the same or other racks significantly decreasing the risk of losing access to or losing data.

3.4. Commodity Hardware
^^^^^^^^^^^^^^^^^^^^^^^

StorPool **supports drives and servers in a vendor-agnostic manner**, allowing you to avoid vendor lock-in. This allows the use of commodity hardware, while preserving reliability and performance requirements. Moreover, unlike RAID, StorPool is drive agnostic – you can mix drives of various types, make, speed or size in a StorPool cluster.

3.5. Shared Block Device
^^^^^^^^^^^^^^^^^^^^^^^^

StorPool **provides shared block devices** with semantics identical to a shared iSCSI or FC disk array.

3.6. Co-existence with hypervisor software
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

StorPool **can utilize repurposed existing servers and can co-exist with hypervisor software on the same server**. This means that there is no dedicated hardware for storage, and growing an IaaS cloud solution is achieved by simply adding more servers to the cluster.

3.7. Compatibility
^^^^^^^^^^^^^^^^^^

StorPool **is compatible with 64-bit Intel and AMD based servers**. We support all Linux-based hypervisors and hypervisor management software. Any Linux software designed to work with a shared storage solution such as an iSCSI or FC disk array will work with StorPool. StorPool guarantees the functionality and availability of the storage solution at the Linux block device interface.

3.8. CLI interface and API
^^^^^^^^^^^^^^^^^^^^^^^^^^

StorPool **provides an easy to use yet powerful command-line interface (CLI) tool** for administration of the data storage solution. It is simple and user-friendly, making configuration changes, provisioning and monitoring fast and efficient.

StorPool also provides a RESTful JSON API, and python bindings exposing all the available functionality, so you can integrate it with any existing management system.

3.9. Reliable Support
^^^^^^^^^^^^^^^^^^^^^

StorPool comes with **reliable dedicated support**:

* remote installation and initial configuration by StorPool's specialists;
* 24x7 support;
* live software updates without interruption in the service.

4. Hardware Requirements
------------------------

All distributed storage systems are highly dependent on the underlying hardware. There are some aspects that will help achieve maximum performance with StorPool and are best considered in advance. Each node in the cluster can be used as server, client, iSCSI target or any combination; depending on the role, hardware requirements vary.

4.1. Minimum StorPool cluster
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- 3 industry-standard x86 servers;
- any x86-64 CPU with 4 threads or more;
- 32 GB ECC RAM per node (8+ GB used by StorPool);
- any hard drive controller in JBOD mode;
- 3x SATA3 hard drives or SSDs;
- dedicated 10GE LAN.

4.2. Recommended StorPool cluster
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- 5 industry-standard x86 servers;
- IPMI, iLO/LOM/iDRAC desirable;
- Intel Nehalem generation (or newer) Xeon processor(s);
- 64GB ECC RAM or more in every node;
- any hard drive controller in JBOD mode;
- dedicated dual 25GE or faster LAN;
- 2+ NVMe drives per storage node.

4.3. How StorPool relies on hardware
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

4.3.1. CPU
""""""""""

When the system load is increased, CPUs are saturated with system interrupts. To avoid the negative effects of this, StorPool's server and client processes are given one or more dedicated CPU cores. This significantly improves overall the performance and the performance consistency.

4.3.2. RAM
""""""""""

ECC memory can detect and correct the most common kinds of in-memory data corruption thus maintains a memory system immune to single-bit errors. Using ECC memory is an essential requirement for improving the reliability of the node. In fact, StorPool is not designed to work with non-ECC memory.

4.3.3. Storage (HDDs / SSDs)
""""""""""""""""""""""""""""

StorPool **ensures the best drive utilization**. Replication and data integrity are core functionality, so RAID controllers are not required and all storage devices might be connected as JBOD. All hard drives are journaled on an NVMe drive similar to Intel Optane series. When write-back cache is available on a RAID controller it could be used in a StorPool specific way in order to provide power-loss protection for the data written on the hard disks. This is not necessary for SATA SSD pools.

4.3.4. Network
""""""""""""""
StorPool **is a distributed system** which means that the network is an essential part of it. Designed for efficiency, StorPool combines data transfer from other nodes in the cluster. This greatly improves the data throughput, compared with access to local devices, even if they are SSD or NVMe.

4.4. Software Compatibility
^^^^^^^^^^^^^^^^^^^^^^^^^^^

4.4.1. Operating Systems
""""""""""""""""""""""""

- Linux (various distributions)
- Windows and VMWare, Citrix Xen through standard protocols (iSCSI)

4.4.2. File Systems
"""""""""""""""""""

**Developed and optimized for Linux, StorPool is very well tested on CentOS, Ubuntu and Debian**. Compatible and well tested with ext4 and XFS file systems and with any system designed to work with a block device, e.g. databases and cluster file systems (like GFS2 or OCFS2). StorPool can also be used with no file system, for example when using volumes to store VM images directly. StorPool is compatible with other technologies from the Linux storage stack, such as LVM, dm-cache/bcache, and LIO.

4.4.3. Hypervisors & Cloud Management/Orchestration
"""""""""""""""""""""""""""""""""""""""""""""""""""

- KVM
- LXC/Containers
- OpenStack
- OpenNebula
- OnApp
- CloudStack
- Any other technology compatible with the Linux storage stack.
