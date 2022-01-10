# PFC Watchdog by hardware\
# Table of Content
# List of Tables
# About this Manual
# Scope
This document describes the high level design of PFC watchdog by hardware on Broadcom TH and TD3 platforms.
# Definitions/Abbreviation
##### Table 1  Definitions
|Definitions/Abbreviation|Description|
|----|----|
|PFC_WD SW|PFC watchdog detection and recovery implemented by software, such as ACL or ZeroBuffer|
|PFC_WD HW|PFC watchdog  detection and recovery implemented by hardware|

# Introduction
PFC watchdog is designed to detect and mitigate PFC storm received for each port.

PFC pause frame is used in lossless Ethernet to pause the link partner from sending packets. Such back-pressure mechanism could propagate to the whole network and cause the network stop forwarding traffic. 

PFC watchdog is to detect abnormal back-pressure caused by receiving excessive PFC pause frames, and mitigate such situation by disable PFC caused pause temporarily. PFC watchdog has three function blocks, i.e. detection, mitigation and restoration.
* Storm Detection: Detect the PFC pause storm if it keeps happening for certain time for a non-drop class.
* Storm Mitigation: Restore the Storm by dropping all packets of the non-drop class. 
* Storm Restoration: PFC watchdog will continue counting the PFC frames received on the queues. If there is no PFC frame received over restoration_time period. Then, re-enable the PFC on the queue and stop dropping packets if the previous mitigation was drop.

Currently, there are 2 lossless queues in SONiC by dafault. The default actions for PFC mitigation is dropping packets at both ingress and egress stage.
On Broadcom platform, the packet drop is implemented by ACL.
* Ingress stage

    Create a shared ACL table for all lossless queues to match `TC` and `IN_PORT`, and drop any received packets.
* Egress stage

    Create individual ACL tables for different lossless queues to match `TC`, and drop any egressed packets.

In Gemini scenario, the downstream traffic from T1 to standby ToR is forwarded to active ToR via a IP tunnel. To ensure a lossless transmit and avoid consuming the volume of current lossless queues, another lossless queue is required.
Limited by TCAM resources, we are not able to add another PFC watchdog since another lossless queue requires another EGRSS ACL table to drop packets. Therefore, we are going to add a new mechanism to implement PFC_WD by hardware.

![Lossless queues for Gemini](./pfc_1.png "Lossless queues between Active ToR and Standby Tor")


# Questions
1. Why can we only have up 2 software watchdog? Is there any other limitations besides TCAM?
2. Read & clear counter
3. 
