The Network Simulator of the LEO satellite with NDN stack
================================
## Overview
The goal is to simulate the NDN traffic on LEO satellites with accurate delays and mobility. The core simulation is based on the modified ndnSIM version of ns-3 (https://github.com/named-data-ndnSIM/ns-3-dev & https://github.com/named-data-ndnSIM/ndnSIM), and the satellite topology, net devices, and channels are taken from Hypatia (https://github.com/snkas/hypatia). However, putting ndnSIM's and Hypatia's components together does not simply work because:
Both act as modules of different versions of ndnSIM. Several patches are needed for compatibility.
The Default NDN Stack (NFD, ndn-cxx, and additional ns-3 functionality) of ndnSIM do not support the nature of satellite topology.
Our approach is to identify the common ground, extract the reusable pieces of the codes, and implement the bridge between the two.

## Brief Specification of the Topology
1) There are two types of nodes: Satellite and Ground Station.
2) There are two types of links: GSL (Ground Station <-> Satellite) and ISL (Satellite <-> Satellite).
3) GSL link is assumed to be broadcast due to the poor performance of the laser from the LEO orbit to the ground.
4) ISL link is assumed to be a point-to-point laser.
5) Each Satellite node has five net devices: one GSL and four ISLs. This is a default value that can be changed.
6) Each Ground Station has one net device: one GSL
7) Each ISL channel connects two Satellite's ISL net devices.
8) Only one GSL channel connects all Satellite's GSL net devices and Ground Station's GSL net devices.
9) The mobility model calculates the delay from a distance between the nodes' positions and the speed of light, regardless of the channel types.

## NDN Stack and Faces
1) Instead of IP Interface, NDN has Faces.
2) Each Face has precisely two components: upper-level Link Service and lower-level Transport.
3) Link Service (in this case, generic-link-service) handles congestion control, packet encoding, managing transmission queue, etc.
4) Transport (in this case, ndn-multicast-net-device-transport) brings the packet to the other end of the net device. Transport is created from a net device.
5) The forwarding component (e.g., FIB table) forwards the name to the Face. The Transport layer of that Face will send the packet to the assigned net device.
6) Implementing a custom Transport is unnecessary for ISL channels since ndnSIM's FibHelper helps connect two nodes on a point-to-point channel. For the GSL channel, the implementation can be tricky because, unlike IP, Transport only knows the "name." Sending Interest forward and receiving Data back complicates the decision of the Transport.

## Modifications/Adoptions of Hypatia's code:
1) Remove the IP-related components (e.g., Arbiter, ARP Cache, and other IP congestion controls). Basic-sim module is still used for utility functions (e.g., exp-util.cc).
2) Fully adopt GSL and ISL channels.
3) Require a small patch for GSL and ISL net devices, such as adding the NDN protocol number.
4) Adopt most of the import functions, including forwarding state and topology data.

## Changes to the base ns-3 code
1) Install and fix missing dependencies caused by different versions of external modules and ns-3.
2) Implement a custom Transport to support GSL multicast.
3) Implement a custom NDN StackHelper class to support different types of net devices and channels.
4) Apply Hypatia's mobility model to the NDN Stack for calculating delays.
5) Extends some of the NFD and ndnSIM components to add functionalities for satellite topology.
6) Internal modules are only modified for compatibility issues.
