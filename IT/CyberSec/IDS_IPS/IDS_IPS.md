---
tags:
  - IT
  - IT/tech
  - IT/CS/util/IDS-IPS
type: tech-concept
Additional:
Abbreviation: Intrusion Detection (Prevention) System
---
---
## Overview
**IDS** — Intrusion Detection System
**IPS** — Intrusion Prevention System

**Signature-based IDS Tools**
With a signature-based IDS, aka knowledge-based IDS, there are rules or patterns of known malicious traffic being searched for. Once a match to a signature is found, an alert is sent to your administrator. These alerts can discover issues such as known malware, network scanning activity, and attacks against servers.

**Anomaly-based IDS Tools**
With an anomaly-based IDS, aka behavior-based IDS, the activity that generated the traffic is far more important than the payload being delivered. An anomaly-based IDS tool relies on baselines rather than signatures. It will search for unusual activity that deviates from statistical averages of previous activities or previously seen activity. For example, if a user always logs into the network from California and accesses engineering files, if the same user logs in from Beijing and looks at HR files this is a red flag.

**Network-based intrusion detection systems (NIDS)** operate by inspecting all traffic on a network segment in order to detect malicious activity. With NIDS, a copy of traffic crossing the network is delivered to the NIDS device by mirroring the traffic crossing switches and/or routers.

A NIDS device monitors and alerts on traffic patterns or signatures. When malicious events are flagged by the NIDS device, vital information is logged. This data needs to be monitored in order to know an event happened. By combining this information with events collected from other systems and devices, you can see a complete picture of your network’s security posture. Note that none of the tools here correlate logs by themselves. This is generally the function of a Security Information and Event Manager (SIEM).

**Host-based intrusion detection systems (HIDS)** work by monitoring activity occurring internally on an endpoint host. HIDS applications (e.g. antivirus software, spyware-detection software, firewalls) are typically installed on all internet-connected computers within a network, or on a subset of important systems, such as servers. This includes those in public cloud environments.

HIDS search for unusual activities by examining logs created by the operating system, looking for changes made to key system files, tracking installed software, and sometimes examining the network connections a host makes.

HIDS have grown far more complex and perform a variety of useful security functions and will continue to grow. This includes modern Endpoint Response (EDR) capabilities.

If your organization has a compliance mandate, such as for PCI DSS, HIPAA, or ISO 27001, then you may require HIDS to demonstrate file integrity monitoring as well as active threat monitoring.
### Purpose
**Purpose**: to detect and prevent intrusions 
### Key responsibilities
- Local machine ( #IT/CS/util/IDS-IPS/HIDS  ) security and logging 
- Local network ( #IT/CS/util/IDS-IPS/NIDS  ) traffic security and logging
### Representation
- **HIDS**
	- OSSEC
	- Samhain labs
	- OpenDPL
- **NIDS**
	- [[SNORT]]
	- [[Suricata]]
	- Bro Network Security Monitor
#### Comparison
![[Pasted image 20260803134940.png]]
