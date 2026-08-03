---
tags:
  - IT/CS/util/IDS-IPS
  - IT/CS/util/IDS-IPS/NIDS
type: cybersec-utility
Technology related to: "[[IDS_IPS]]"
Source:
---
## Overview
**IDS** — Intrusion Detection System
**IPS** — Intrusion Prevention System

**Purpose**: to detect and prevent intrusion

More information is listed in the [[IDS_IPS|IDS/IPS technology note]].
### Key features
- **Multi-Engine Operation (IDS / IPS / NSM):**
    - **IDS (Intrusion Detection System):** Passive network monitoring and alert generation.
    - **IPS (Intrusion Prevention System):** Active inline threat blocking (via `netfilter` / `NFQUEUE`).
    - **NSM (Network Security Monitoring):** Deep packet logging and protocol extraction for forensic analysis.
      
- **Multi-Threading Architecture:**
    - Native multi-threaded engine designed to scale across multiple CPU cores for high-throughput, multi-gigabit traffic processing.
      
- **High performance** - multi-threaded, scalable code base
- **Multipurpose Engine** - NIDS, NIPS, NSM, offline analysis, etc.
- **Cross-platform support** - Linux, Windows, macOS, OpenBSD, etc.
- Modern TCP/IP support including a scalable flow engine, full IPv4/IPv6, TCP streams, and IP packet defragmentation
- **Protocol parsers** - packet decoding, application layer decoding
- **HTTP engine** - HTTP parser, request logger, keyword match, etc.
- Autodetect services for portless configuration
- **Lua** scripting (LuaJIT)
- **Application-layer logging** and analysis, including TLS/SSL certs, HTTP requests, DNS requests, and more
- Built-in **hardware acceleration**
- **File extraction**
## Installation
To install **Suricata** we simply need to run a set of commands:
### Importing
1. Update the packet manager: `$    sudo apt update`
2. Import Suricata from an official repo: `$    sudo apt install suricata -y`
3. Install **JQ**:  `$    sudo apt install suricata jq`
4. Download the latest signature rule base: `$    sudo suricata-update`
5. Inspect build info: `$    sudo suricata --build-info`
6. Look at the daemon status: `$    sudo systemctl status suricata`

![[Pasted image 20260803100137.png]]

### Configuration
We need to change the `HOME_NET` variable in a `/etc/suricata/suricata.yaml` file according to the network configuration:
1. Define the network **IP range**:
   ![[Pasted image 20260803101032.png|659]]
   
2. Update the `HOME_NET` accordingly:![[Pasted image 20260803100610.png]]
   
3. Make sure the `interface` variable in `af-packet` matches the interface used:
   ![[Pasted image 20260803101217.png]]
   
4. `cluster-id:99` – assigns a cluster identifier to the packets which is useful for categorizing and monitoring packets flow in order to determine which packets belong to which cluster
   ![[Pasted image 20260803101421.png]]
   
5. `cluster-type: cluster_flow` – specifies the type of clustering that will be applied to the packets. In this case, it is `cluster_flow`, which indicates that the clustering is based on flow.
   ![[Pasted image 20260803101517.png]]
   
6. `defrag: yes` – enables packet **defragmentation**:
   ![[Pasted image 20260803101604.png]]
   
7. `tpacket-v3: yes` – specifies the use of `TPACKET_V3`, which is a packet capture method on Linux systems. `TPACKET_V3` offers improved performance and efficiency compared to previous versions.
   ![[Pasted image 20260803102616.png]]
   
8. `use-mmap:yes` Enables the ring buffer mechanism (PACKET_MMAP) via pcap/socket. This allows network packets to be transferred from the kernel to user space directly through shared memory, minimizing unnecessary data copying and reducing the CPU load.
   
   **ACCORDING TO AI:** In modern versions of Suricata, the ring buffer is already used by default for the `af-packet` driver at the Linux kernel level, so explicitly specifying this directive within the `af-packet` block is most often a legacy requirement from older guides. 
   
   So that we will explicitly *just in case* cause of an assignment:
   ![[Pasted image 20260803102736.png]]

## Running Suricata
In this section various ways to interact with Suricata are listed
#### Starting
At first we need to deploy the daemon:`sudo systemctl restart suricata`
![[Pasted image 20260803104221.png]]

#### Logs inspection
A section primarily about log files
##### Functioning
We can inspect Suricata functionality logs: `sudo tail /var/log/suricata/suricata.log`
![[Pasted image 20260803104335.png]]

##### Performance
We can also inspect performance metrics and counters: `sudo tail -f /var/log/suricata/stats.log`
![[Pasted image 20260803104430.png]]

This log entry reports the launch of the Suricata engine and the creation of various types of threads. Specifically, it indicates:

- one thread was created for processing packets (**W - Workers**)
- one thread for processing outgoing packets (**FM - Flow Managers**)
- one thread for parsing incoming packets (**FR - Flow Recyclers**)

It also indicates that the engine was successfully started.

##### Summary
Quick summary is provided in `fast.log` file, making it readable quickly for a reliable intrusion-response or reaction:`sudo tail -f /var/log/suricata/fast.log`
![[Pasted image 20260803104630.png]]

#### Logs testing
We can test one of the security features of Suricata by running`curl http://testmynids.org/uid/index.html`
![[Pasted image 20260803104709.png]]

Yes, this is essentially a `testmynids.org/uid/index.html` website request. It simply contains a single string `uid=0(root) gid=0(root) groups=0(root)`. 

Most standard Suricata rule sets (such as **Emerging Threats**) include a signature that looks for the text `uid=0(root)` in incoming HTTP traffic, as this is *an indication of a successful RCE*.

When this command is executed, Suricata should immediately generate an alert in `fast.log` or `eve.json`. Which it does:
![[Pasted image 20260803120207.png]]

#### JQ
We can utilize [[JSON]] parser util –  [[jq]] in order to dynamically select properly categorized logs among the whole bunch of them: `$ sudo tail -f /var/log/suricata/eve.json| jq 'select(.event_type=="stats")'`

![[Pasted image 20260803105136.png]]
### Suricata rules
**A Suricata rule** — is an atomic instruction (signature) that defines which network traffic should be considered suspicious and how to respond to it. Since the traffic gets encrypted more end more, many *false-positives* might occur. To mitigate that use modern rules by running `$ sudo suricata-update`.

>**Purpose**: To find a pattern match in a data stream.
>**Function**: Used by **IDS/IPS** systems to detect attacks, anomalies, and malicious activity.
>**Structure**: Specifies an action, a header, and search conditions (options).
#### Structure
A rule consists of three key parts:
- **Action (alert)**  — What to do when a match occurs (alert, drop, pass, reject).
  
- **Header** **(**`http $HOME_NET any -> $EXTERNAL_NET any`**)** — Who is sending traffic, where it’s going, and how (**protocol**, **IP**, **ports**, **direction ->**).
  
- **Rule Options** **(**`(...)`**)** — What to look for inside the packet and metadata (`signature, content patterns, message msg, identifier sid:123`).

Mnemonic:
`[ACTION] [HEADER] ([RULE OPTIONS])`
#### Location
The rules are located within a directory `/var/lib/suricata/rules` in a number of files:
- `local.rules` — user rules standard file which apparently came from Snort
- `custom.rules` — user rules (frequently used instead of `local.rules` for Suricata)
- `suricata.rules` — official rules are downloaded here
#### Examples
An example of rules
```
alert http any any -> any any (msg: "do not read gossip during work"; flow: to_client, established; classtype: policy-violation; sid: 10001; rev: 1;)

alert icmp any any -> any any (msg: "finally pinged"; sid: 10002; rev: 1;)
```
### Monitoring
`suricata -c /etc/suricata/suricata.yaml -i enp0s3 -v`
![[Pasted image 20260803111705.png]]

>[!quote]- The command
>`suricata -c /etc/suricata/suricata.yaml -i enp0s3 -v`
>##### In a nutshell
>The command activates the IDS engine to listen on the enp0s3 interface using the settings from `suricata.yaml` and displays the entire process directly on the screen:
>
>- `-c /etc/suricata/suricata.yaml` — specifies the path to the main configuration file (networks, rules, logs).
>
>- `-i eth0` — specifies the network interface (enp0s3) on which Suricata will intercept and analyze packets.
>
>- `-v `(verbose) — enables detailed console output (shows initialization steps, loaded rules, and system messages).

![[Pasted image 20260803112207.png]]
## Assignments

**List:**
- [x] According to the lecture materials, install, configure, and start the Suricata IDS.
- [x] Stop Suricata, add two custom rules based on the example in the presentation materials. It is recommended to modify the alert message text. 
- [x] Start Suricata, ensure that the additional rules are loaded, and test their functionality. 
- [x] Provide screenshots with a brief explanation for each step.

## Steps
Step by step demonstration of task completion.
### Starting Suricata
Is listed within the [[#Running Suricata]] section.
### Custom rules addition
For this assignment (in order not to crash the entire thing) let's stop the daemon running:
```bash
$ sudo systemctl stop suricata  

$ sudo systemctl status suricata
```

So that it becomes **inactive**:
![[Pasted image 20260803122721.png]]

Let's then create a custom rules file within the standard `/var/lib/suricata/rules/` directory:
```bash
$ sudo vim /var/lib/suricata/rules/local.rules
```

Let's create 5 custom rules, which are aimed to detect some diverse patterns:
- Outbound **HTTP GET** detecting – just to illustrate the point of the possibility itself
- Outbound query containing the key word *"ATTACK"* – to show that tracking query content is possible
- Bidirectional **ICMP** protocol – to track whether we are getting scanned
- Inbound **SSH** connections – to track those
- Outbound **RSYNC** file copying – there are various tools (like `scp`), but this one is used just for instance

The rules listing screenshot:
![[Pasted image 20260803130336.png]]

A slightly more convenient way to list the custom rules:
```
# 1. A custom rule for HTTP requests detection
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"CUSTOM DETECT: Outbound HTTP GET Request"; flow:established,to_server; http.method; content:"GET"; sid:1000001; rev:1;)

# 2. A custom rule for a custom substring detection "ATTACK"
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"CUSTOM SUSPICIOUS: Trigger Rule Query in URI"; flow:established,to_server; http.uri; content:"ATTACK"; fast_pattern; sid:1000002; rev:1;)

# 3. A custom rule to detect any ICMP pinging
alert icmp any any -> any any (msg:"CUSTOM DETECT: ICMP Ping Detected"; itype:8; sid:1000003; rev:1;)

# 4. A custom rule that notifies on an attempt of establishing an SSH connection
alert ssh $EXTERNAL_NET any -> $HOME_NET 22 (msg:"CUSTOM SSH: Inbound SSH Connection Request"; flow:to_server,established; sid:1000004; rev:2;)

# 5. A rule that detects an attempt of copying files with RSYNC (for instance) 
alert tcp $HOME_NET any -> $EXTERNAL_NET 873 (msg:"CUSTOM RSYNC: Outbound Sync Session Initiated"; flow:to_server,established; content:"@RSYNCD:"; depth:8; sid:1000005; rev:1;)
```

So that we now need to update a `suricata.yaml` file in order for Suricata to take care of the custom rules. By running  `$ sudo vim /etc/suricata/suricata.yaml` we can access the file itself and then we need to find `default-rule-path` directive and modify `rule-files` by adding a just created one to the list:
![[Pasted image 20260803124319.png]]

We now may run a `suricata.yaml` syntax comparability check with `$ sudo suricata -T -c /etc/suricata/suricata.yaml`:
![[Pasted image 20260803130319.png]]

Since the check has been completed successfully, we may now move on to the deployment stage.
### Restarting and resting
Let's run Suricata as a process by `$ sudo suricata -c /etc/suricata/suricata.yaml -i eth0 -v`: 
![[Pasted image 20260803131628.png]]

Alternatively, we may start the daemon with `$ sudo systemctl start suricata`. Basically, it is a matter of choice, but there are some key differences:
- Running it as a process gives us an opportunity to quickly inspect it status. It also *captures* the shell, so we will need a separate one for testing.
- Running is as a daemon would be performed in a background. We will need to use `systemctl` and `journalctl` to access it direct output.
Eventually, both approaches are applicable, so feel free to use the more convenient one.

Let's test both *GET method* rule and *keyword rule*:
![[Pasted image 20260803131223.png]]

As we can see, both rules have been triggered by the same action (as expected).

Let's ping the machine by IP and see if the *ICMP* rule is triggered:
![[Pasted image 20260803131347.png]]

So that we have confidently verified that the custom rules are applied.

As it usually getting passed to the `journalctl` (when run as a daemon), there is a log with an amount of alerts triggered:
![[Pasted image 20260803131852.png]]