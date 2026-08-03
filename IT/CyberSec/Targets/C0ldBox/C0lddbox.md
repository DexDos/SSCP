---
tags:
  - IT/CS/CTF
type: cybersec-target
Source:
  - https://www.vulnhub.com/entry/colddbox-easy,586/
CS themes:
  - WordPress
  - Privilege Escalation
  - Reverse shell
  - Port scanning
---
## Overview
**Purpose**: to reinforce the basics
**Target OS**: Linux Ubuntu
**Attack OS**: Kali Linux
**Difficulty**: #IT/CS/CTF/Easy 

### Key notes
- Runs on a [Ubuntu](Ubuntu.md) VM. 
- Uses [CMS](CMS.md) [WordPress](WordPress.md) for running purposes[cite: 10].
- Has a hidden SSH on a non-standard port[cite: 10]
### The main Approach
**The main approach**[cite: 10]
1. **Reconnaissance**[cite: 10]
	1. Discovering ip[cite: 10]
	2. [Port scanning](Port%20scanning.md)[cite: 10]
		1. Define the running software versions[cite: 10]
		2. Define [CVEs](CVE.md) for running software[cite: 10]
	3. Fuzz directories[cite: 10]
	4. Use [wpscan](wpscan.md)[cite: 10]
		1. Scan addons[cite: 10]
		2. Scan users (enumeration)[cite: 10]
2. **Gaining access**[cite: 10]
	1. [Brute force](Brute%20force.md) authentication page[cite: 10]
		1. Use `wpscan` to run a brute force with `rockyou` against users list[cite: 10]
	2. Login[cite: 10]
	3. Observing webpages[cite: 10]
		1. [Forensics](Forensics.md)[cite: 10]
		2. Observe privileges[cite: 10]
		3. Discover available attack vectors[cite: 10]
		4. Detect a way to capitalize on a CMS[cite: 10]
	4. Deploy reverse shell[cite: 10]
	5. Upgrade the shell to a stable state (gaining TTY / PTY)[cite: 10]
3. **Privilege escalation**[cite: 10]
	1. find usernames in `/home` or list `/etc/passwd`[cite: 10]
	2. find files which belong to users or contain key words: `password`, `uname`, `credentials` and so on[cite: 10]
	3. try to find credentials[cite: 10]
	4. `su` to switch to user who has permissions to use `sudo` with vulnerable commands[cite: 10]
	5. gain access to the **root shell**[cite: 10]
4. **Persistence**[cite: 10]
	1. Create a [Backdoor](Backdoor.md) to provide a hidden, secure and reliable access to the machine[cite: 10]
### Weaknesses to leverage
- [WordPress issues](WordPress%20issues.md) related to the WP Auth page[cite: 10].
- [Reverse shell](reverse%20shell.md) deployment via WEB CMS[cite: 10].
- [SSH users enumeration](SSH.md) is another possibility:[cite: 10]
- `xmlrpc.php` is running $\rightarrow$ vulnerable to `multicall` brute force by server runtime[cite: 10]
- Apache server `logrotate`  privilege escalation vulnerability: **CVE-2022-1348**[cite: 10]
- Search for not encrypted files which may contain password variations[cite: 10]
	- Create a custom wordlist if necessary and run another *brute force*[cite: 10]

## Deployment of an instance
To deploy an instance we barely need to perform a sequence of dedicated actions[cite: 10].
1. Download the `.ova` file from the source (stated in properties of this file)[cite: 10]. 
2. Create a new machine in **VirtualBox**  and use the obtained file to create an instance[cite: 10].
   ![Pasted image 20260723183530.png](../../../../Cache/IMGs/Pasted%20image%2020260723183530.png)
3. It is optional to regenerate a new MAC address[cite: 10]
   ![Pasted image 20260723183826.png](../../../../Cache/IMGs/Pasted%20image%2020260723183826.png)
   It is also worth mentioning that standard VirtualBox MAC starts with the `08:00:27` prefix so that we are able to identify the instance among other devices in a subnet[cite: 10].
4. In addition we are able to manage network properties so that we know the way to access the machine within the subnet[cite: 10]. Make sure that the proper Network adapter is set (Wi-Fi / Ethernet)[cite: 10].
5. The rest of the process is self-explanatory[cite: 10]. 

## Cases

| link                                                                   | description                                                                 |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [Standard pentest case](1.md)                                           | Just a simple walkthrough with basic vulnerabilities exploited              |
| [Hammering the machine with exploits](2.md)                            | Is done "the hard way": modified the machine to show diverse exploits usage |

## Flags

| received in            | flag                                                 |
| ---------------------- | ---------------------------------------------------- |
| `/home/c0ldd/user.txt` | RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ== |
| `/root/root.txt`       | wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=     |