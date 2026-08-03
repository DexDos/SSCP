---
tags:
  - IT/CS/CTF
type: cybersec-target
Source:
  - https://owasp.org/www-project-webgoat/
CS themes:
  - CS-lab
---
## Overview
**Purpose**: to reinforce the basics
**Target OS**: Docker image `webgoat/webgoat`
**Attack OS**: Kali Linux
**Difficulty**: #IT/CS/CTF/Easy 

### What is WebGoat?
Welcome *username*!

WebGoat is a deliberately insecure application that allows interested developers just like you to _test vulnerabilities_ commonly found in Java-based applications that use common and popular open source components.

Now, while we in no way condone causing intentional harm to any animal, goat or otherwise, we think learning everything you can about security vulnerabilities is essential to understanding just what happens when even a small bit of unintended code gets into your applications.

What better way to do that than with your very own scapegoat?

Feel free to do what you will with him. Hack, poke, prod and if it makes you feel better, scare him until your heart’s content. Go ahead, and hack the goat. We promise he likes it.

Thanks for your interest!

### Key notes
The WebGoat is a set of instructive tasks featuring versatile vulnerabilities which websites might (commonly) contain. Among the topics covered:
- Broken access control
- Cryptographic failures
- Injection
- Security misconfiguration
- Vuln & outdated components
- Identity & Auth failure
- Software & data integrity
- Security & login failures
- Server-side request forgery
- Challenges

### Approaches
The main approach is to carefully read instructions and creatively study the requests sent by the server. Sometimes assignments are simply composed in a bit odd way ("*it is gonna take some guesses*") so that the luck needed to "guess" is kinda insane. 

## Deployment of an instance
### Separately
The machine requires at least two ports for two services: WebGoat and WebWolf – the target and attack utility website respectively. Consequently, the machine needs to be provided with at least two ports for them to start properly: `8080` and `9090`.

Let's write a docker compose file so we may use this container later.
>[!quote]+ The command
>`$ vim docker-compose.yml`
>
>A `docker-compose.yml` is a file which is used by [Docker CLI](../../../../Docker.md) to deploy versatile containers. We may start this container with both `docker-compose` (outdated, written in python) and `docker compose` (faster, more reliable, not a separate util, but embedded into Docker CLI, which is written in GoLang).

The file needs no be composed in a certain syntax of `.yml` and contain specific keys:

![docker compose](../../../../Cache/IMGs/Pasted%20image%2020260730172035.png)

In this case we have defined a single docker container with two ports (`8080` and `9090`) forwarded. We have also specified a rule `restart: always`, which makes Docker CLI restart this container every time it is getting shut down, restarted and so on. Practically, this means that this container will be started every time the machine starts. In our case, we run WebGoat on a local private server (Ubuntu server LTS v24) so that the WebGoat will be started subsequently.

We may then run `docker compose up -d` to start the container (Docker reads the file and deploys services specified) and check availability with `docker ps` command:
![docker ps](../../../../Cache/IMGs/Pasted%20image%2020260801130201.png)

After a while we may study logs by running ` $ docker compose logs webgoat` command:
![docker compose logs](../../../../Cache/IMGs/Pasted%20image%2020260730173629.png)

Note that in logs there is this direction: browse to `http://:8080/WebGoat`. We are going to need it somewhat later

Then we are going to need an IP-address. To retrieve it we need find out our IP range with `$ ip a` command:
![ip a](../../../../Cache/IMGs/Pasted%20image%2020260730173958.png)

Assuming that our connection to the network is provided with Ethernet connection, the adapter we are looking for would be `eth0`or  `ens33` one. For now we need to obtain the local IP address of the server we will need to use the `netdiscover command`

> [!quote]+ The command
> `$ sudo netdiscover -r 192.168.31.0/24`
> 
> ##### The flags used
> - `-r` — specifies the IP <u>range</u> for search
> 
> More about [netdiscover](../../../../netdiscover.md)

The output of the command:
![netdiscover output](../../../../Cache/IMGs/Pasted%20image%2020260730173827.png)

We have obtained three IP-addresses as candidates for being target's address. Let's break them down:
- `192.168.31.1` — the *1* in the last octet gives us information that this IP is related to the network address and is being monitored by the router.
- `192.168.31.69` — provided that the *MAC Vendor / Hostname* column refers to **Compal**, we may suppose that stated MAC-address is related to the notebook hosting both machines (The target and the attack machines). Compal is a huge corporation manufacturer which ships chips and electronics all over the place. If the last IP doesn't convince us that it belongs to the machine, we'll go back to this option. So far it doesn't seem to be suspicious. 
- `192.168.31.104` — This option has a very specific MAC address. It's MAC starts with the prefix `00:0c:29` which directly indicates that it belongs to a **VM Ware**. Thus we may complete our search, because we have found the needed IP, presumably.

Now that we have successfully discovered an IP of the WebGoat, we now need to browse it in a browser. Basically, the direction (in logs) was to browse for `http://:8080/WebGoat`. This could have been the way for us, had we been hosting it on the Kali itself. The browser would have automatically added the loopback to the query (`http://127.0.0.1:8080/WebGoat` / `http://localhost:8080/WebGoat`). However, we are hosting it on a separate server, so we need to add an IP of the server. 

So that: `http://:8080/WebGoat` $\rightarrow$ `http://192.168.31.104:8080/WebGoat`:

![webgoat page](../../../../Cache/IMGs/Pasted%20image%2020260730173432.png)

### Locally
Alternatively, we may run WebGoat locally:

```
$ docker run --name webgoat -it -p 127.0.0.1:8080:8080 -p 127.0.0.1:9090:9090 webgoat/webgoat
```
![docker run](../../../../Cache/IMGs/Pasted%20image%2020260801152312.png) 

## Registration
We also need to register: 

![registration 1](../../../../Cache/IMGs/Pasted%20image%2020260801152505.png) 
Fill out the registration form: 

![registration 2](../../../../Cache/IMGs/Pasted%20image%2020260801152542.png) 
We may now proceed to the assignments: 

![assignments](../../../../Cache/IMGs/Pasted%20image%2020260801152609.png) 
## HTTP Proxy
Let's turn on our HTTP proxy in order to study HTTP traffic. 
### FoxyProxy

We are using **FoxyProxy** one, but you are free to use any you like: 

![foxyproxy](../../../../Cache/IMGs/Pasted%20image%2020260801154037.png) 
Proxy configuration: 

![proxy config](../../../../Cache/IMGs/Pasted%20image%2020260801162147.png) 
> [!question]- How does an http proxy work >Let's break this down. Usually, when we surf the internet our browser establishes a connection with it using some port on our local machine: >![direct connection](../../../../Cache/IMGs/Pasted%20image%2020260801160219.png) >This procedure grans a way to study the traffic with some kind of software like **Wireshark**. Unfortunately, by simply listening the traffic we are unable to perform many kinds of MITM attacks due to the matter of fact that we cannot stop a packet (intercept) and spoof some value before it reaches the server (or the client). Of course we may attempt jamming the channel, but this comes with some drawbacks. >We may establish an HTTP Proxy connection instead: >![proxy connection](../../../../Cache/IMGs/Pasted%20image%2020260801161248.png) >So that we are able to control the traffic flow and perform various attacks in the way neither server, nor the client can spot our intrusion. >The proxy itself is a technology that redirects traffic. This technology can be used to send HTTP Connect method, thus redirecting traffic completely via proxy server. However, in this case we simply redirecting in and outbound traffic to the port specified. By default, Burp Suite listens on the 8080 port. Unfortunately, the WebGoat server listens on the exact same port. To evade any kind of conflict we have specified `8123` port for FoxyProxy configuration. 

### Burp Suite
We have picked the **Burp Suite** to be the program which will be intercepting our http traffic. However there are various programs to use instead. For example **ZAP**. Let's configure the Burp Proxy:

![burp proxy 1](../../../../Cache/IMGs/Pasted%20image%2020260801162634.png) 

Then we need to specify the socket which we will be listening: `loopback:8123`: 

![burp proxy 2](../../../../Cache/IMGs/Pasted%20image%2020260801162812.png) 
Afterwards we can turn the proxy profile on and test our proxy: 

![burp proxy test](../../../../Cache/IMGs/Pasted%20image%2020260801163203.png) 

### Common problems 
There is a common problem which happens when users try to use Burp against resources hosted locally. Burp simply wouldn't see the traffic. According to some resources, this happens so the burp won't explode with a great number of traffic which is being sent over the standard interface among the machine services. To bypass this limitation we simply need to modify host aliases in `/etc/hosts` so that burp treats our resource not as a local one: 
![hosts file](../../../../Cache/IMGs/Pasted%20image%2020260801173207.png) 

By doing so we now may see the traffic by requesting `http://localhost.com:8080/WebGoat/login` page: 

![burp traffic](../../../../Cache/IMGs/Pasted%20image%2020260801173134.png) 
Alternatively we can simply use Burp built-in browser: 

![burp browser 1](../../../../Cache/IMGs/Pasted%20image%2020260801173327.png) 

The result is pretty much the same: 

![burp browser 2](../../../../Cache/IMGs/Pasted%20image%2020260801173302.png) 
## Cases 
There are two introductory sections: *introduction* and *general*. - **Introduction** lists some basic facts about WebGoat and WebWolf further usage. - **General** dives a bit into the networking, dev tools and other stuff. These sections are pretty basic do not cover any specific information. They are aimed to introduce the user to some fundamental concepts and tools which are necessary for further assignments.

| link | status | description |
| --- | --- | --- | 
| [Broken Access Control](./1.md) | **done \| uploaded** | |
| [Cryptographic failures](./2.md) | done \| not reachable | | 
| [Injection](./3.md) | done \| not reachable | | 
| [Security Misconfiguration](./4.md) | done \| not reachable | | 
| [Vuln & Outdated Components](./5.md) | to be done | | 
| [Identity & Auth Failure](./6.md) | to be done | | 
| [Software & Data Integrity](./7.md) | to be done | | 
| [Security & Login Failures](./8.md) | to be done | | 
| [Server-side Request Forgery](./9.md) | to be done | | 

## Flags 

| received on | flag |
| --- | --- | 
| | |
