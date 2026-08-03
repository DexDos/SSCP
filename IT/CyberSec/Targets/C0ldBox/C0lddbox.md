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
- Runs on a [[Ubuntu]] VM. 
- Uses [[CMS]] [[WordPress]] for running purposes.
- Has a hidden SSH on a non-standard port
### The main Approach
**The main approach**
1. **Reconnaissance**
	1. Discovering ip
	2. [[Port scanning]]
		1. Define is the running software versions 
		2. Define [[CVE|CVEs]] for running software 
	3. Fuzz directories
	4. Use [[wpscan]]
		1. Scan addons
		2. Scan users (enumeration)
2. **Gaining access**
	1. [[Brute force]] authentication page
		1. Use `wpscan` to run a brute force with `rockyou` against users list 
	2. Login
	3. Observing webpages 
		1. [[Forensics]]
		2. Observe privileges
		3. Discover available attack vectors
		4. Detect a way to capitalize on a CMS
	4. Deploy reverse shell
	5. Upgrade the shell to a stable state (gaining TTY / PTY)
3. **Privilege escalation**
	1. find usernames in `/home` or list `/etc/passwd`
	2. find files which belong to users or contain key words: `password`, `uname`, `credentials` and so on
	3. try to find credentials
	4. `su` to switch to user who has permissions to use `sudo` with vulnerable commands
	5. gain access to the **root shell**
4. **Persistence**
	1. Create a [[Backdoor]] to provide a hidden, secure and reliable access to the machine
### Weaknesses to leverage
- [[WordPress issues]] related to the WP Auth page.
- [[reverse shell|Reverse shell]] deployment via WEB CMS.
- [[SSH|SSH users enumeration]] is another possibility: 
- `xmlrpc.php` is running $\rightarrow$ vulnerable to `multicall` brute force by server runtime
- Apache server `logrotate`  privilege escalation vulnerability: **CVE-2022-1348**
- Search for not encrypted files which may contain password variations
	- Create a custom wordlist if necessary and run another *brute force*

## Deployment of an instance
To deploy an instance we barely need to perform a sequence of dedicated actions.
1. Download the `.ova` file from the source (stated in properties of this file). 
2. Create a new machine in **VirtualBox**  and use the obtained file to create an instance.
   ![[Pasted image 20260723183530.png]]
3. It is optional to regenerate a new MAC address
   ![[Pasted image 20260723183826.png]]
   It is also worth mentioning that standard VirtualBox MAC starts with the `08:00:27` prefix so that we are able to identify the instance among other devices in a subnet.
4. In addition we are able to manage network properties so that we know the way to access the machine within the subnet. Make sure that the proper Network adapter is set (Wi-Fi / Ethernet).
5. The rest of the process is self-explanatory. 
## Cases

| link                                                                   | description                                                                 |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [[IT/CyberSec/Targets/C0ldBox/1\|Standard pentest case]]               | Just a simple walkthrough with basic vulnerabilities exploited              |
| [[IT/CyberSec/Targets/C0ldBox/2\|Hammering the machine with exploits]] | Is done "the hard way": modified the machine to show diverse exploits usage |

## Flags

| received in            | flag                                                 |
| ---------------------- | ---------------------------------------------------- |
| `/home/c0ldd/user.txt` | RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ== |
| `/root/root.txt`       | wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=     |


