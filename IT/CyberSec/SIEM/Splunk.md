---
tags:
  - IT
  - IT/tech
  - IT/CS/util/SIEM
type: tech-representation
Additional:
Abbreviation:
The concept:
  - "[[SIEM]]"
---
---
## Overview
**Splunk** is a popular [SIEM](SIEM) tool that is known for its powerful search and analytics capabilities. It can collect, store, and analyze data from a wide variety of sources, including logs, events, and metrics. Splunk is also highly scalable, making it a good choice for organizations of all sizes. Some of the key features of Splunk include:
### Key features
- **Powerful search and analytics**: Splunk's search language is very powerful and can be used to query and analyze data from any source.

- **Real-time data analysis**: Splunk can analyze data in real time, which can help organizations to detect and respond to security threats more quickly.

- **Wide range of integrations:** Splunk can integrate with a wide range of security and IT tools, which can help organizations to create a unified view of their security posture.

## Free trial
There is an opportunity to experience a **free trial** on the [official website](https://www.splunk.com/en_us/download/splunk-cloud/cloud-trial.html). 
### Registration

1. Fill out the form and create an account
2. Verify an email
3. Wait for a confirmation email with credentials:

![Pasted image 20260803211044.png](../../../Cache/IMGs/Pasted%20image%2020260803211044.png)


### Getting started
4. Log in

![Pasted image 20260803211127.png](../../../Cache/IMGs/Pasted%20image%2020260803211127.png)

5. Change the temporary password immediately 

![Pasted image 20260803211415.png](../../../Cache/IMGs/Pasted%20image%2020260803211415.png)

6. Read and accept terms and policies
7. The supporting materials were downloaded from `SoftServe`, thus the demonstration relies on them
8. Upload data with *Add Data*

![Pasted image 20260803211820.png](../../../Cache/IMGs/Pasted%20image%2020260803211820.png)

9. Select (or drag & drop) files. In this case the `tutorialdata (2).zip` file

![Pasted image 20260803211925.png](../../../Cache/IMGs/Pasted%20image%2020260803211925.png)

10. In the *Next* section select `Segment in path` and type `1` in the input box:

![Pasted image 20260803212004.png](../../../Cache/IMGs/Pasted%20image%2020260803212004.png)

11. Review the result:

![Pasted image 20260803213355.png](../../../Cache/IMGs/Pasted%20image%2020260803213355.png)

12. There is a quiz assignment to be done afterwards: [Splunk practice task for quiz.pdf](../../../Cache/IMGs/Splunk%20practice%20task%20for%20quiz.pdf)

### Completion
Proceed to "*Search & Reporting*":

![Pasted image 20260803214917.png](../../../Cache/IMGs/Pasted%20image%2020260803214917.png)

We may now execute search queries:

![Pasted image 20260803214954.png](../../../Cache/IMGs/Pasted%20image%2020260803214954.png)

> **Tip**: It's a best practice to use short time ranges in your searches because a shorter time range returns results faster and uses fewer resources. Adjust the time using the time range dropdown or by using time modifiers in your search.

When Splunk indexes data, it attaches fields to each event. These fields become part of the searchable index event data. This helps security analysts easily search for and find the specific data they need. Now that you've run your first query, examine the search results and the fields.
For each event the fields are:
- `host`
- `source`
- `sourcetype`
Under `SELECTED FIELDS`, examine the same fields:

![Pasted image 20260803215825.png](../../../Cache/IMGs/Pasted%20image%2020260803215825.png)

Examine the field values by clicking on the field under` SELECTED FIELDS`

![Pasted image 20260803220214.png](../../../Cache/IMGs/Pasted%20image%2020260803220214.png)

You should observe the following: 
- `host`: The host field specifies the name of the network host from which the event originated. In this search there are five hosts: 
	- **mailsv** - Buttercup Games' mail server. Examine events generated from this host. 
	- **www1** - This is one of Buttercup Games' web applications. 
	- **www2** - This is one of Buttercup Games' web applications. 
	- **www3** - This is one of Buttercup Games' web applications. 
	- **vendor_sales** - Information about Buttercup Games' retail sales.
	  
	  ![Pasted image 20260803220248.png](../../../Cache/IMGs/Pasted%20image%2020260803220248.png)
	  
- `source`: The source field indicates the file name from which the event originates. You should identify eight sources. Notice `/mailsv/secure.log`, which is a log file that contains information related to authentication and authorization attempts on the mail server.
  
  ![Pasted image 20260803220328.png](../../../Cache/IMGs/Pasted%20image%2020260803220328.png)
  
- `sourcetype`: The `sourcetype` determines how data is formatted. You should observe three `sourcetypes`. 
  
  ![Pasted image 20260803220501.png](../../../Cache/IMGs/Pasted%20image%2020260803220501.png)


Examine `secure-2`:

![Pasted image 20260803220533.png](../../../Cache/IMGs/Pasted%20image%2020260803220533.png)

Because you've been tasked with exploring any failed **SSH** logins for the root account on the mail server, you'll need to *narrow the search* *results* for events from the mail server. Under `SELECTED FIELDS`, click `host` and click `mailsv`. 

![Pasted image 20260803220908.png](../../../Cache/IMGs/Pasted%20image%2020260803220908.png)

Notice that a new term has been added to the search bar: `index=main host=mailsv`. The search results have narrowed to over 9000 events that are generated by the mail server. 

>**How to pass Splunk quiz?** Brief description. Solve every Use Case in Splunk Cloud, get results and put them as an answer in proper quiz question. Could be done in parallel way or just write down results of Use Case Searches.

#### Use Case1 (example) `[visible]`

Search for a failed login for root Now that you've narrowed your search results to events generated by the mail server, continue to narrow the search to locate any failed SSH logins for the root account.

1. Clear the search bar. 
2. Enter `index=main host=mailsv fail* root` into the search bar. 
   
   This search expands on the search from the previous task and searches for the keyword `fail*`. The wildcard tells Splunk to expand the search term to find other terms that contain the word fail such as *failure*, *failed*, etc. Lastly, the keyword *root* searches for any event that contains the term `root`. 
   
3. Set up Date and Time Range 

![Pasted image 20260803221443.png](../../../Cache/IMGs/Pasted%20image%2020260803221443.png)

*(ALL SEARCHES SHOULD USE THE SAME Date and Time Range)*

> **Answer:** Found number of events is a quiz answer for the proper question: `41`

![Pasted image 20260803221603.png](../../../Cache/IMGs/Pasted%20image%2020260803221603.png)

> **41**  – confirmed
#### Use Case2
Search for a *failed* login for *root* on host “`www1`” (`**`) at the same Date and Time Range.

![Pasted image 20260803221749.png](../../../Cache/IMGs/Pasted%20image%2020260803221749.png)

> **67**
#### Use Case3
Search for a *failed* login for *root* on host “`www2`” (`**`) at the same Date and Time Range.

![Pasted image 20260803222057.png](../../../Cache/IMGs/Pasted%20image%2020260803222057.png)

> **57**
#### Use Case4 (example) `[visible]`
Search for a *HTTP* responses status “*Client side error*”

> **Answer:**`sourcetype=access_* status=40*` Check number of Events(439)

![Pasted image 20260803222330.png](../../../Cache/IMGs/Pasted%20image%2020260803222330.png)

> **439** – confirmed

#### Use Case5 
Search for a *HTTP* responses status “*Client side error*” from “`www1`” host. Check number of Events(`***`)

![Pasted image 20260803222725.png](../../../Cache/IMGs/Pasted%20image%2020260803222725.png)

> **122**

#### Use Case6 
Search for a *HTTP* responses status “*Client side error*” from “`www3`” host. Check number of Events(`***`)

![Pasted image 20260803222908.png](../../../Cache/IMGs/Pasted%20image%2020260803222908.png)

>**141**

#### Use Case7 (example) `[visible]`

Search for a *HTTP* responses status “*Server-side error*” 
>**Answer:**`sourcetype=access_* status=50*`Check number of Events(306)

![Pasted image 20260803223213.png](../../../Cache/IMGs/Pasted%20image%2020260803223213.png)

> **306** – confirmed

#### Use Case8 
Search for a *HTTP* responses status “*Server-side error*” from “`www2`” host. Check number of Events(`**`)

![Pasted image 20260803223345.png](../../../Cache/IMGs/Pasted%20image%2020260803223345.png)

> **88**

#### Use Case9 
Search for a *HTTP* responses status “*Server-side error*” from “`www3`” host Check number of Events(`***`)

![Pasted image 20260803223500.png](../../../Cache/IMGs/Pasted%20image%2020260803223500.png)

> **113**

#### Use Case10 (example) `[visible]`
Search for a *SSH* *accepted* login attempts 

>**Answer:** `sourcetype="secure-2" accept*` Check number of Events(229)

![Pasted image 20260803223658.png](../../../Cache/IMGs/Pasted%20image%2020260803223658.png)

> **229** – confirmed

#### Use Case11 
Search for a *SSH* *failed* login attempts. Check number of Events(`****`)

![Pasted image 20260803223908.png](../../../Cache/IMGs/Pasted%20image%2020260803223908.png)

> **5034**

#### Use Case12 
Search for a *SSH* *failed* login attempts for host “`www2`” Check number of Events(`****`)

![Pasted image 20260803224059.png](../../../Cache/IMGs/Pasted%20image%2020260803224059.png)

> **1289**
#### Use Case13 
Search for a *SSH* *accepted* login attempts for host “`www3`”. Check number of Events(`**`)

![Pasted image 20260803224320.png](../../../Cache/IMGs/Pasted%20image%2020260803224320.png)

> **60**

#### Use Case14 
Search for a *SSH* *accepted* login attempts for host “`mailsv`”. Check number of Events(`**`)

![Pasted image 20260803224544.png](../../../Cache/IMGs/Pasted%20image%2020260803224544.png)

> **39**


## Concept
### How it works
When Splunk indexes data, it attaches fields to each event. These fields become part of the searchable index event data. This helps security analysts easily search for and find the specific data they need. Now that you've run your first query, examine the search results and the fields.
For each event the fields are:
- `host`
- `source`
- `sourcetype`

![Pasted image 20260803215802.png](../../../Cache/IMGs/Pasted%20image%2020260803215802.png)

### features



