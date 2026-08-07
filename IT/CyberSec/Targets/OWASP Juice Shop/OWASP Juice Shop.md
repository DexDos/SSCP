---
tags:
  - IT/CS/CTF
type: cybersec-target
Source:
  - https://owasp.org/www-project-juice-shop/
CS themes:
  - CS-lab
---
## Overview
**Purpose**: reinforce the basics
**Target OS**: docker container 
**Attack OS**: Kali Linux
**Difficulty**: #IT/CS/CTF/Easy #IT/CS/CTF/Medium #IT/CS/CTF/Hard 

### Key notes
OWASP Juice Shop is probably the most modern and sophisticated insecure web application! It can be used in security trainings, awareness demos, CTFs and as a guinea pig for security tools! Juice Shop encompasses vulnerabilities from the entire [OWASP Top Ten](https://juice-shop.github.io/www-project-top-ten) along with many other security flaws found in real-world applications!
### Approaches
- Find *Score board*
- Look for assignments
- Study the assignment theme and proceed for accomplishment
- Use hints if necessary
### Weaknesses to leverage
The Juice shop is a deliberately vulnerable app, containing dozens of weaknesses. The structure:
![Pasted image 20260802114322.png](../../../../Cache/IMGs/Pasted%20image%2020260802114322.png)

## Deployment of an instance
Deployment is usually done in various ways:
- `bkimminich/juice-shop` docker image
- `https://github.com/juice-shop/juice-shop.git` git repo
- `juice-shop-<version>_<node-version>_<os>_x64.zip` packet installation

Command for docker container deployment:
> `docker run --name juice-shop -it -p 3000:3000 bkimminich/juice-shop`

Ultimately, we are getting pretty much the same result:
![Pasted image 20260802132120.png](../../../../Cache/IMGs/Pasted%20image%2020260802132120.png)
## Cases
The assignments accomplished are listed here

| link                            | difficulty | description                                                                                                             |
| ------------------------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------- |
| [Score Board](1.md)             | 1/6        | Utilize dev tools in order to find a hidden page                                                                        |
| [DOM XSS](2.md)                 | 1/6        | A basic copy paste skills usage to perform an XSS attack with a payload given                                           |
| [Sensitive data exposure](3.md) | 1/6<br>4/6 | Utilizing `ffuf` to eventually retrieve a document from a hidden (disallowed in `robots.txt`) route. <br>**Easter Egg** |
| [Admin login](4.md)             | 2/6        | Basic sqli                                                                                                              |
| [User Credentials](5.md)        | 4/6        | Slightly advanced sqli                                                                                                  |
| [[6\|Ephemeral Accountant]]     | 4/6        | SQLite schema retention, sqli login as not existing user                                                                |
## Flags
The flags captured are listed here

| received on | flag |
| ----------- | ---- |
|             |      |