# pfSense
## Overview
pfSense is an open-source firewall and router platform based on FreeBSD. It is a powerful, flexible, and highly configurable solution that can be used to secure and protect the network infrastructure.

## HA Architecture
The pfSense firewall is deployed in a high-availability (HA) configuration to ensure continuous operation and failover protection. The HA setup consists of two pfSense firewalls running in active-passive mode. The primary firewall handles all traffic, and the secondary firewall takes over in case of a failure.

To achieve HA, we need two virtual IP addresses (VIPs) for the WAN and LAN interfaces. And a CARP (Common Address Redundancy Protocol) IP address for the synchronization of the configuration between the primary and secondary firewalls.
