.. _ceph-nvmeof:

======================
 Ceph NVMe-oF Gateway
======================

The NVMe-oF Gateway presents an NVMe-oF target that exports
RADOS Block Device (RBD) images as NVMe namespaces. The NVMe-oF protocol allows
clients (initiators) to send NVMe commands to storage devices (targets) over a
TCP/IP network, enabling clients without native Ceph client support to access
Ceph block storage.  

Each NVMe-oF gateway consists of an `SPDK <https://spdk.io/>`_ NVMe-oF target
with ``bdev_rbd`` and a control daemon. Ceph’s NVMe-oF gateway can be used to
provision a fully integrated block-storage infrastructure with all the features
and benefits of a conventional Storage Area Network (SAN).

.. ditaa::
                  Cluster Network (optional)
                 +-------------------------------------------+
                 |             |               |             |
             +-------+     +-------+       +-------+     +-------+
             |       |     |       |       |       |     |       |
             | OSD 1 |     | OSD 2 |       | OSD 3 |     | OSD N |
             |    {s}|     |    {s}|       |    {s}|     |    {s}|
             +-------+     +-------+       +-------+     +-------+
                 |             |               |             |
      +--------->|             |  +---------+  |             |<----------+
      :          |             |  |   RBD   |  |             |           :
      |          +----------------|  Image  |----------------+           |
      |           Public Network  |    {d}  |                            |
      |                           +---------+                            |
      |                                                                  |
      |                      +--------------------+                      |
      |   +--------------+   | NVMeoF Initiators  |   +--------------+   |
      |   |  NVMe‐oF GW  |   |    +-----------+   |   | NVMe‐oF GW   |   |
      +-->|  RBD Module  |<--+    | Various   |   +-->|  RBD Module  |<--+
          |              |   |    | Operating |   |   |              |
          +--------------+   |    | Systems   |   |   +--------------+
                             |    +-----------+   |
                             +--------------------+

==============================
 High Availability / GW groups
==============================
High Availability (HA) provides I/O and control path redundancies for the host initiators. High Availability is also sometimes referred to as failover and failback support. The redundancy that HA creates is critical to protect against one or more gateway failures. With HA, the host can continue the I/O with only the possibility of performance latency until the failed gateways are back and functioning correctly.

NVMe-oF gateways are virtually grouped into gateway groups and the HA domain sits within the gateway group. An NVMe-oF gateway group supports eight gateways. Each NVMe-oF gateway in the gateway group can be used as a path to any of the subsystems or namespaces that are defined in that gateway group. HA is effective with two or more gateways in a gateway group.

High Availability is enabled by default. To use High Availability, a minimum of two gateways and listeners must be defined. 

It is important to create redundancy between the host and the gateways. To create a fully redundant network connectivity, be sure that the host has two Ethernet ports that are connected to the gateways over a network with redundancy (for example, two network switches).

The HA feature uses the Active/Standby approach for each namespace. Using Active/Standby means that at any point in time, only one of the NVMe-oF gateways serve I/O from the host to a specific namespace. To properly use all NVMe-oF gateways, each namespace is assigned to a different load-balancing group. The number of load-balancing groups is equal to the number of NVMe-oF gateways in the gateway group.

With HA, if an NVMe-oF gateway fails, the initiator continues trying to connect. The amount of time that it tries to connect for depends on what is defined for the initiator. For more information about defining the reconnect time for the initiator and general configuration instructions, see Configuring the NVMe-oF gateway initiator.

.. toctree::
  :maxdepth: 1

  Requirements <nvmeof-requirements>
  Configuring the NVME-oF Target <nvmeof-target-configure>
  Configuring the NVMe-oF Initiators <nvmeof-initiators>
