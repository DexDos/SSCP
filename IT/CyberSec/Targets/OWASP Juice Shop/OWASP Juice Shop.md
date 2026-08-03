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
OWASP Juice Shop is probably the most modern and sophisticated insecure web application! It can be used in security trainings, awareness demos, CTFs and as a guinea pig for security tools! Juice Shop encompasses vulnerabilities from the entire [OWASP Top Ten](https://juice-shop.github.io/www-project-top-ten) along with many other security flaws found in real-world applications!
### Approaches
- Find *Score board*
- Look for assignments
- Study the assignment theme and proceed for accomplishment
- Use hints if necessary
### Weaknesses to leverage
The Juice shop is a deliberately vulnerable app, containing dozens of weaknesses. The structure:
![[Pasted image 20260802114322.png]]

## Deployment of an instance
Deployment is usually done in various ways:
- `bkimminich/juice-shop` docker image
- `https://github.com/juice-shop/juice-shop.git` git repo
- `juice-shop-<version>_<node-version>_<os>_x64.zip` packet installation

Command for docker container deployment:
> `docker run --rm -p 3000:3000 bkimminich/juice-shop`

Ultimately, we are getting pretty much the same result:
![[Pasted image 20260802132120.png]]
## Cases
The assignments accomplished are listed here

| link                                                                | difficulty | description                                                                                                             |
| ------------------------------------------------------------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------- |
| [[IT/CyberSec/Targets/OWASP Juice Shop/1\|Score Board]]             | 1/6        | Utilize dev tools in order to find a hidden page                                                                        |
| [[IT/CyberSec/Targets/OWASP Juice Shop/2\|DOM XSS]]                 | 1/6        | A basic copy paste skills usage to perform an XSS attack with a payload given                                           |
| [[IT/CyberSec/Targets/OWASP Juice Shop/3\|Sensetive data exposure]] | 1/6<br>4/6 | Utilizing `ffuf` to eventually retrieve a document from a hidden (disallowed in `robots.txt`) route. <br>**Easter Egg** |
| [[4\|Admin login]]                                                  | 2/6        | Basic sqli                                                                                                              |
| [[5\|User Credentials]]                                             | 4/6        | Slightly advanced sqli                                                                                                  |
## Flags
The flags captured are listed here

| received on | flag |
| ----------- | ---- |
|             |      |


