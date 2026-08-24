---
tags:
  - IT
  - IT/tech
  - protocol/transfer/mail
type: tech-concept
Additional:
Abbreviation:
---
---
## Overview

A standard protocol for **sending** mail is [[SMTP]]:

![Pasted image 20260815220500.png](../../../../Cache/IMGs/Pasted%20image%2020260815220500.png)

Originally, SMTP was serving to send mail from sender's MUA to the respondent's MUA directly and it is still possible to do so, bypassing all these fancy MDA/MRA agents. Unfortunately, such a way of usage is vastly inconvenient because it needs both sender and recipient to stay online. Subsequently, recipients might not be available at some point which will cause SMTP failures and potential SPOOL overflow problems, casting a MUA readiness requesting necessity alongside. To solve this kind of problem international mail infrastructure implemented mail brokers (MDA / MRA), which either give full access to the mail stored on a broker server ([[IMAP]]), or passing it to the recipient when they are online ([[POP3]]).

Although the infrastructure is already robust enough, it is still vulnerable to a number of versatile exploitation techniques like [[phishing]]. To mitigate any risks the world came up with some set of defensive measures, which are being implemented all over the place: [[DKIM]], [[SPF]] and [[DMARC]].

## Validation concept
The validation is meant to be executed on a first MX relay of the target domain infrastructure. It may be done with diverse set of analytical approaches like body analysis, email elements inspection, spam database check and specific headers analysis. After the validation process has ended, MX relay adds an evaluation of a suspect value (e.g., hasn't passed `spf`, but passed `dmarc` and `dkim` ), validated with a kind of panic mode value result `Received-SPF` or `Authentication-Results` header to the SMTP message.

To set up a mail server we essentially need four DNS records:

![Pasted image 20260821181220.png](../../../../Cache/IMGs/Pasted%20image%2020260821181220.png)

| record | type  | designation                                                                                                                                               |
| ------ | ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SPF    | `TXT` | **Sender Policy Framework** — a mechanism designated to validate sender by its DNS relation to the domain used                                            |
| DKIM   | `TXT` | **DomainKeys Identified Mail** — a mechanism designated to approve or refute the message integrity via signature and hash validation                      |
| DMARC  | `TXT` | **Domain-based Message Authentication, Reporting & Conformance** — domain based validation, which evaluates final decision based on SPF and DKIM results  |

The validation process needs to retrieve respective DNS records alongside:

![Pasted image 20260821212531.png](../../../../Cache/IMGs/Pasted%20image%2020260821212531.png)

### Inbound MX Relay 
An initial point of a corporate subnetwork which has direct access to the WAN and is responsible for traffic filtering, routing and so on. It is used to perform SPF, DKIM and DMARC validation before passing mail to further MDA nodes:

![Pasted image 20260821211845.png](../../../../Cache/IMGs/Pasted%20image%2020260821211845.png)


### MX
Specifies the relative subdomain which is responsible for mail handling.

Example of an MX record from [Cloudflare docs](https://www.cloudflare.com/learning/dns/dns-records/dns-mx-record/):

| example.com | record type: | priority: | value:                | TTL   |
| ----------- | ------------ | --------- | --------------------- | ----- |
| @           | MX           | 10        | mailhost1.example.com | 45000 |
| @           | MX           | 20        | mailhost2.example.com | 45000 |
Using Google Admin Toolbox to send a DNS-question about `MX` record of a google.com domain:

![Pasted image 20260823190023.png](../../../../Cache/IMGs/Pasted%20image%2020260823190023.png)


### SPF

> **Relevant standard:** [RFC 7208](https://datatracker.ietf.org/doc/html/rfc7208).

RFC requires a **single** `TXT` DNS-record, which starts with `v=spf1`. Having multiple been provided, the validity would be evaluated by the first one, although it should cause an error. Missing `v=` part is also 

#### Validation procedure
According to RFC 7208, the SPF validation is settled on the inbound MX-relay in order to validate the initial sender securely and not get a spoofed `Received-SPF` header in. The check starts with `check_host()` function which is used for an algorithm reference in order to illustrate the canonical procedure. All implementations must mimic behavior and semantic execution results, but not the function itself. 

The function takes three arguments: `ip`, `domain` and `sender`:

- `ip` — the IP address of the SMTP client that is emitting the mail
- `domain` — a domain fraction of the received `MAIL FROM` or `HELO` (`EHLO`) identity (`...@domain`)
- `sender` — a SMTP `MAIL FROM` (or `HELO`) identity, which will be used as `RETURN PATH` if needed.

![Pasted image 20260821205254.png](../../../../Cache/IMGs/Pasted%20image%2020260821205254.png)

It is important to send the initial SMTP packet via TCP directly to the MX relay, cuz otherwise `HELO` (`EHLO`) domain would differ, so the further validation could end with some issues related to domains mismatch, thus if some service validates by `HELO` as well spf check will result with softfail / fail:

```SMTP
HELO relay-domain
250 OK mx-relay
MAIL FROM sender-identity@sender-domain
250 2.1.0 Ok
...
```

To mitigate these risks, such relay may resend the SMTP as they were the sender domain, but it comes with both pros and cons. 

Examples of spf records:

```
v=spf1 ip4:198.51.100.25 include:_spf.google.com ~all

v=spf1 a mx ip4:203.0.113.0/24 -all

v=spf1 redirect=_spf.holding-company.com

v=spf1 -all
```

These mechanisms specified should be stored in a single DNS TXT record starting with `v=spf1`. They are used in left-to-right order. The validation procedure relies on so-called qualifiers (optional; `+` is the default option) and mechanisms. 

![Pasted image 20260821213626.png](../../../../Cache/IMGs/Pasted%20image%2020260821213626.png)

##### Qualifiers
Optional part of a rule which stands at the beginning: `+all`. They state the way the match should be interpeted:
-  `-` — fail
-  `+` — pass (default one)
-  `~` — softfail ( perform additional checks )
-  `?` — neutral ( domain neither confirms nor refutes the validity )

##### Mechanisms
Necessary part of a rule. Indicates that should be taken as a reference for the validation:
- `all` — anything. Is usually used the least to fail or softfail requests which did not match any previous rules
- `include` — specifies another domain which `spf` record should be taken for the further validation (a way of delegation for subdomains)
- `a` — allows sending emails from the address specified in the `A` and/or `AAAA` records of the sender’s domain
- `mx` — allows sending emails from the address specified in the MX records of the domain
- `ptr` — (*deprecated*) performs a reverse DNS query to retrieve the sender-domain `ptr` record and check if it matches the specified one. 
- `ip4`, `ip6` — specify the IP (or CIDR range) reference to validate by
- `exists` — is used to construct an arbitrary domain to perform a DNS A query by (may use macros: `exists:%{ir}.%{l1r+-}._spf.%{d}` which validates by such domain constructed `1.2.0.192.someuser._spf.example.com`). If any A record is returned, this mechanism matches.

##### Modifiers
Modifiers are name/value pairs that provide additional information. Modifiers always have an "=" separating the name and the value:

```
redirect=<domain>
exp=<domain>
<name>=<macro>
```

- `redirect` — fully delegates validation to the `spf` of the domain specified. If none matches the validation stops (whereas `include` mechanism doesn't stop validation on the specified domain's `spf`, if none matches).
    
- `exp` — (explanation) If `check_host()` results in a `fail` due to a mechanism match (such as `-all`), and the `exp` modifier is present, then the explanation string returned is computed. Say we were to come across such modifier value: `exp=explain._spf.%{d}`. It is then taken (macros are executed) for a DNS TXT query target (`explain._spf.domain`). The strings fetched from the resource record are concatenated with no spaces, and then treated as an explain-string, which is macro-expanded
    
- `macros` — When evaluating an SPF policy record, certain character sequences are intended to be replaced by parameters of the message or of the connection. These character sequences are referred to as "macros". It is better to watch [source](https://datatracker.ietf.org/doc/html/rfc7208#section-7) for the further explanation.

#### Results
There are several results possible which also handle errors might occur: 

| result      | qualifier | description                                              | actions               |
| ------------| --------- | -------------------------------------------------------- | --------------------- |
| `None`      |           | no spf record found                                      | accept with no trust  |
| `Neutral`   | `?`       | The domain neither approves not refutes the sender       | at your discretion    |
| `Pass`      | `+`       | The domain approves the sender                           | accept                |
| `Fail`      | `-`       | The domain refutes the sender                            | reject                |
| `Softfail`  | `~`       | A soft fail, IP didn't match any rule                    | accept with suspicion |
| `Temperror` |           | DNS didn't respond                                       | try again later       |
| `Permerror` |           | DNS spf config record could not be correctly interpreted | accept with suspicion |

### DKIM

> **Relevant standard:** [RFC 6376](https://datatracker.ietf.org/doc/html/rfc6376).

Hash algorithm versions:
- `sha256` 
- `sha1` 

Digital signature algorithm versions:
- `RSA` — minimum 2048 bit length recommended
- `Ed25519` — added in **RFC 8463**

To do this, we need to generate a key pair:
```bash
$ openssl genrsa -out private.pem 2048 #generate a 2048-bit private key
```


```bash
$ openssl rsa -pubout -in private.pem -out public.pem #derive the public key from the private key
```

Alternatively, you can use an online service.

Next, you need to specify the path to the private key in the mail server’s configuration file (it’s best to consult the documentation for this) and the public key in DNS.

An example of the records is

```
mail._domainkey.your.tld TXT "v=DKIM1; a=rsa-sha256; t=s; p=<public key>"
```

where

`mail` — the **selector**. You can specify multiple records with different selectors, where each record will have its own key. This is used when multiple servers are involved. Each server has its own key
`v` — the DKIM version, which always takes the value `v=DKIM1`
`k` — key type (`k=rsa` or `k=ed25519`) 
`a` — specifies supported signature and hashing algorithms (`-` is a delimiter)
`p` — public key, encoded in `base64`
`t` — Flags:
- `t=y` — test mode. These records differ from unsigned ones and are intended solely for tracking results
- `t=s` — means that the record will be used only for the domain to which the record applies; not recommended if subdomains are used

Possible values:

`h` — signed header fields (plain-text, but see description; REQUIRED). A colon-separated list of header field names that identify the
      header fields presented to the signing algorithm. The field MUST contain the complete list of header fields in the order presented to the signing algorithm.
`s` — The type of service using DKIM. Accepts the values` s=email` (email) and `s=*` (all services). The default is `*`.
`;` — separator.
`i` —  the AUID on behalf of which the SDID is taking responsibility
`l` — body length count

Another example of a header:
```txt
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; d=chess.com; h=content-transfer-encoding:content-type:date:from:mime-version:subject: reply-to:list-id:list-unsubscribe:list-unsubscribe-post:to:cc:content-type: date:feedback-id:from:subject:to; s=s1; t=1786642452; bh=CAS4oFyI7dSG995B14hBadLxvU0JDO+1Jt2wYCnmG1M=; b=GclmDhqlo/RyyGjHhue7jVSSKx0RbmKRYsLE2uvchYqdwIZxV3WmQMImfbXKHnyU0MFS /VPZgd79kGFmZX+lDGOXZNE3wRbb65D/07g0ajTVYmti1YozOG8k+hc5gT8wyOh4kVgZAk a1lGGMYxQGxeVPefDWsoZGpdSSkTCfrdR+HDwWdY/Bk2JKRwDmbetWMhcoZMGr7mpsBLB4 ongjKFwqXXwIw/VXCvyc9E2ZoQ2B40eKZWLMOi/HPYdpGCfig5lzeGxIq7HVNhBrTR+y56 dmUhJR7f9c7jST/cou5y0sAtXQGoyAxbPmnnIp33smwgVfQ+PRDD5HTUZQ6yx/UA==
```

Here:
`h` — list of headers which take part in a signature validation
`bh` — body hash needed to verify integrity. Since `a=rsa-sha256`, the hashing algorithm used is `sha256`:
	`bh = base64( sha256( canonicalized_body ))`
	`canonicalized_body` — everything transferred within SMTP `data` command, canonicalized to the common syntax
`b` — the signature:
	`b = base64( RSA_signature( sha256( canonicalized_headers <CR><LF> DKIM-Signature_header )))`

More about [[RSA]], [[ECDSA]] and [[SHA-256]].

Once our server has retrieved an email we need to verify its validity:

![Pasted image 20260824105036.png](../../../../Cache/IMGs/Pasted%20image%2020260824105036.png)

Using the DKIM-Signature header we can look for its `s` parameter to define the selector used. Then we can forge (`<selector>._domainkey.<domain>`) the domain which must have the relative RSA key in its DNS TXT record:
```txt
id 15456 
opcode QUERY 
rcode NOERROR 
flags QR RD RA 
;QUESTION 
s1._domainkey.chess.com. IN TXT 
;ANSWER
s1._domainkey.chess.com. 300 IN CNAME s1.domainkey.u2137297.wl.sendgrid.net. s1.domainkey.u2137297.wl.sendgrid.net. 1800 IN TXT "k=rsa; t=s; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsqgC8kjpzWWtb2TcoQozNlmmr71aQyd5E2Edx3w8JNt05j6k4pPpNybU3BxZ3842YqcqwbGY0nxpB+H5vU4l12tjYSSW9NPpbZNpJ/scGSEdrdDFjpsVqI21iKQ+GwoZixn10kz/RPtjXbG3yBK4fdCHOMm+BoIgMulagf8P4DLMi2QEqCFX2w8vS7dh+DFxJycG8" "Pu3LbS1TYlNcrwWbBwmLMGQCNNU2eicUfWHEFkXRDjBUSvqLMvnsiI7NZmbuBZnYxhzPFnFDvo8aIOrTuzN6kbCnyNKXcOTwYunRDeAlbCTXsMK6fbdHypxKFH8vOiOMFXhypaVJv7edZfGtQIDAQAB"
;AUTHORITY 
;ADDITIONAL
```

It is worth mentioning, that string length in DNS records is limited. To bypass this limitation we can set multiple strings separated with `%20` which are going to be concatenated on retrieval.

Afterwards we perform the exact same canonicalization, integrity check and signature validation.

#### Canonicalization
Needed to bring an email to a common format:
- `From:    <sender> ` to `from: <sender>` — relaxed canonicalization

Reference parameter: `c`.
Delimiter: `/`.
There are rly only 2 options: `simple` and `relaxed`. A `c` parameter value consists of two parts separated by the delimiter. The first part specifies header canonicalization rule and the second stays for body canonicalization.

Only four options possible:
```
c=simple/simple
c=simple/relaxed
c=relaxed/simple
c=relaxed/relaxed
```

`simple` — nothing will be changed
`relaxed`:
- Headers:
	- Header keys (not values) are brought to lowercase 
	- Multiple space (`%20`) chars are replaced with a single one
	- Odd offsets are deleted
	- Each header is ended with `<CR><LF>`
- Body
	- `<CR><LF><CR><LF>` ( header / body delimiter ) and `<CR><LF>.<CR><LF>` ( end of email ) are not included
	- Odd spaces and tabulation are deleted ( end of lines )
	-  Empty lines are simplified to a single `<CR><LF>`.

So say we have retrieved an email:
```
From:      Friend <friend@gmail.com> <CR><LF>
Subject:      Black tea<CR><LF>
<CR><LF>
What a day. Such a wonderful day <CR><LF>
<CR><LF>
<CR><LF>
     <CR><LF>
   <CR><LF>
<CR><LF>
Bye!<CR><LF>
<CR><LF>
   <CR><LF>
<CR><LF>
.<CR><LF>
```

The `simple` canonicalization will leave the message to be bit-to-bit same. Whereas a `relaxed` will convert it to:

```
from: Friend <friend@gmail.com><CR><LF>
subject: Black tea<CR><LF>
<CR><LF>
What a day. Such a wonderful day<CR><LF>
<CR><LF>
<CR><LF>
<CR><LF>
<CR><LF>
<CR><LF>
Bye!<CR><LF>
.<CR><LF>
```

#### Results

According to **RFC 6376**, there are only two results:
- `Pass`
- `Fail`

Although it is [updated](https://datatracker.ietf.org/doc/html/rfc7601#section-2.7.1) with **RFC 7601** which specifies extra responses:

| result      | description                                                                                                                | action                |
| ----------- | -------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| `none`      | No DKIM DNS record                                                                                                         | accept with no trust  |
| `neutral`   | The message was signed, but the signature or signatures contained syntax errors or were not otherwise able to be processed | at your discretion    |
| `pass`      | The domain approves the sender                                                                                             | accept                |
| `fail`      | The domain refutes the sender                                                                                              | reject                |
| `policy`    | Algorithm (`a`) is not supported                                                                                           | at your discretion    |
| `permerror` | DNS spf config record could not be correctly interpreted                                                                   | accept with suspicion |
| `temperror` | DNS didn't respond                                                                                                         | try again later       |
### DMARC

> **Relevant standard:** [RFC 9989](https://datatracker.ietf.org/doc/html/rfc9989).

Since **SPF** checks domain spoofing and **DMARK** is used to check the integrity they wont be able to alert user about an email sent from a malicious domain. Their checks use `MAIL FROM` domain as a reference, which scammer may set up properly:

![Pasted image 20260824123753.png](../../../../Cache/IMGs/Pasted%20image%2020260824123753.png)

DMARC is needed to mitigate these risks and to determine the further procedure based on SPF and DKIM responses. 

A typical record looks like this: `_dmarc.your.tld TXT “v=DMARC1; p=none; rua=mailto:postmaster@your.tld”`. This record does not perform any actions other than generating and sending a report.

Now, let’s take a closer look at the tags:

`v` — version; takes the value `v=DMARC1` 
`p` — rule for the domain. Can take the values none, quarantine, and reject, where
- `p=none` does nothing except generate reports
- `p=quarantine` adds the email to the SPAM folder
- `p=reject` rejects the email

The `sp` tag is responsible for subdomains and accepts the same values as `p`

`aspf` and `adkim` allow you to verify compliance with records and can take the values `r` and `s`, where `r` (relaxed) is a less strict check than `s` (strict).
`pct` controls the percentage of emails to be filtered; for example, `pct=20` will filter 20% of emails.
`rua` — allows you to send daily reports to an email address; for example:` rua=mailto:postmaster@your.tld`. You can also specify multiple email addresses separated by spaces (`rua=mailto:postmaster@your.tld mailto:dmarc@your.tld`).
`ruf` — reports of emails that failed DMARC validation. Otherwise, everything is the same as above.

#### The procedure 
The server needs to take a DATA From domain part and retrieve the TXT record starting with `v=DMARC1`:

![Pasted image 20260824132347.png](../../../../Cache/IMGs/Pasted%20image%2020260824132347.png)

DMARC then checks validity of domains based on `aspf` and `adkim` parameters, which specify whether domains need to match strictly (`s`) or subdomains may also be used (`r`):

![Pasted image 20260824133948.png](../../../../Cache/IMGs/Pasted%20image%2020260824133948.png)

If at least one `DKIM d` or `SPF Mail From` **alligns** `From` then DMARC is approved and mail gets transferred to MDA (Mail Delivery Agent). If none matches, the retrieved from DNS query policy is enforced:
- `none` — no limitations take place. Mail gets delivered. Domain host gets notified by reports
- `quarantine` — email is sent to SPAM
- `reject` — email does not get delivered

Tags for feedbacks usually store email addresses with a `mailto:` prefixes
- `rua` — specifies email to send regular `.xml` reports
- `ruf` — specifies email to send specific reports about failed DMARC
- `rf` — specifies report format to be sent:
	- `afrf` — Authentication Failure Reporting Format ( **RFC 6591** ) the only option available at this point. `rf` parameter is preserved for formats may appear in future.
- `fo` — specifies condition under which report gets generated:
	- `1` — if at least one check fails
	- `0` — if both checks fail
	- `s` — if `spf` check fails 
	- `d` — if `dkim` check fails
- `pct` — specifies the percentage of failed email to which the `dmarc p` policy is applied. Is needed to slowly migrate to a more strict policies or configurations without risking all the email traffic failures. Is deprecated by `t`:
	- `100` — 100%
	- `0` — 0%
- `t` — DMARC policy test mode:
	- `y` — testing mode on:
		- `p=none` — no changes
		- `p=quarantine` — gets sent as if it were `p=none`
		- `p=reject` — gets sent as if it were `p=quarantine`
	- `n` — no test mode. `p` is applied to failed emails

Other security related parameters:
- `sp` — DMARC subdomain policy. By default inherits `p` tag value and behaves exactly the same way based on `aspf` and `adkim` which is `r` (relaxed) by default:
	- `none` 
	- `quarantine`
	- `reject`
- `np` — sets a dedicated policy for non-existent subdomains similarly to `p` and `sp`. Say there was a `Mail From: non-existent.example.com`. If no record found the **RFC 9989** states that the server will need to perform a *Tree Walk* ( [source](https://datatracker.ietf.org/doc/html/rfc9989#section-4.10) ) and to perform `_dmarc.example.com. IN TXT` query, which may contain either less strict limitations (e.g., `p=none`) or no `TXT v=DMARC1` record at all, in which case DMARC check is skipped ( [source](https://datatracker.ietf.org/doc/html/rfc9989#section-4.10.1) ):
  ```
  If the set produced by the DNS Tree Walk contains no DMARC Policy 
  Record (i.e., any indication that there is no such record as opposed 
  to a transient DNS error), Mail Receivers MUST NOT apply the DMARC 
  mechanism to the message.
  ```
  So that, if the main domain `example.com` were to contain `v=DMARC1 ... np=reject`, the DMARC policy would reject an email which has potentially spoofed `Mail From: non-existent.example.com`.
- `psd` — is a marker of a public suffix ( `github.io`, `hosting.com` etc. ). If `psd=y`is found, the server needs to process its DMARC policy and everything depends on `p` tag of that record
	- `y` — yes
	- `n` — no
	- `u` — undefined (default)
#### Results

Say DMARC domain check results with `pass` due to `spf` alignment. For DMARC to result with `pass` we need either `spf` or `dkim` to result with `pass` as well. Any failures are going to be evaluated by `p` parameter: `spf=fail`, `dkim=fail`, `dmarc p=none` $\rightarrow$ an email is going to be transferred. 

There is a cool website [learndmarc](https://www.learndmarc.com/) to test general understanding of concepts and principles of result evaluation:

![Pasted image 20260824155512.png](../../../../Cache/IMGs/Pasted%20image%2020260824155512.png)

## Vulnerabilities 
#TODO
### SPF
- Помилки конфігурації — дозволяють надто широкі діапазони IP відтак можна застосувати proxy
- Експлуатація рекурсивної перевірки ( ліміт 10 DNS запитів ) для того аби замаскувати нелегітимного відправника. Таким чином у репортах не буде відображено spf fail, а лише permerror, що вказує на помилки конфігурації та, знову ж таки в теорії, викликає менше підозр при аналізі.
- RCE, тунелювання, telnet і тп — способи легітимізувати відправлення через фактичну підміну IP адреси відправлення
- Використання публічних поштових MTA relays у яких з якихось причин виставлено `v=spf1 +all` та які переписують листи від імені власного домену. Таким чином, адреса початкового відправника маскується. Можна створити власний тимчасовий варіант подібного сервісу, отримуючи pass у результаті. Таким чином, достатньо виставити у `v=DKIM1 d` легітимний домен та в теорії лист має проходити DMARC.
- telnet підключення до поштового сервісу, пошкодження `MAIL FROM` вмісту для того, аби перевірка відбулася за HELO цього самого поштового сервісу
- Якщо сервіс користується хмарними сервісами в тому числі для відправки повідомлень, через що йому доводиться легітимізувати цілий пул, скажімо Amazon IP адрес, то ми можемо тимчасово орендувати потужності того ж самого Amazon Lambda аби відправляти листи від імені цього сервісу, підробляючи лише domain у `MAIL FROM`.
### DKIM
- Витік чи компрометація ключів
- Підміна DNS та, відповідно, валідація за неоригінальним ключем
- DDoS, використання астрономічно довгих `bh=` значень, некоректні символи та інші шляхи переповнення буферу токенізатора парсера з метою його виходу з ладу у фолбек, сподіваючись, що він налаштований як Fall-Open ( [джерело](https://datatracker.ietf.org/doc/html/rfc6376#section-8.9) ).
- Експлуатація синтаксичних ін'єкцій для низькорівневих парсерів ( для мов де строки не зберігаються зі значенням довжини, а натомість шукається нульовий байт `0x00`), що у листі можна замаскувати під **ascii art**. Таким чином, теоретично, можна перехопити чужий лист та порушити цілісність, при чому проходження DKIM.
- Перехоплення вже відправлених листів для того, аби відправити їх же змінені копії із валідним підписом:
	- В залежності від того, який за послідовністю заголовок обробляється сервісами MTA та MUA можна маніпулювати вмістом доставляючи у лист додаткові заголовки Subject, From і тд.
	- Якщо у початковому листі вказано тег `l=`, то можна не змінювати вказану кількість байт, а додати ін'єкцію із фішинговим вмістом у кінець листа, таким чином валідно проходячи DKIM та DMARC
	- Додавання у елементів зі зміненими MIME. MTA може зчитати, скажімо, нижній заголовок `Content-Type: text/plain`, що додано у криптографічний підпис, відтак валідація пройде. Але можна також вставити вище заголовок `Content-Type: multipart/alternative; boundary="scam_bound"` та додати boundary у тіло, аби MUA відобразив приховану фішингову частину. Це працює бо **RFC 2046** допускає неточності, тож на боці типово толерантних MUA контент відобразиться. Водночас, валідатор DKIM парсер згідно **RFC 6376** зобов'язаний взяти саме нижній заголовок `Content-Type`. Дійсно, цей шлях можна захистити дублюванням `h=Content-Type:Content-Type:...` у конкретному випадку, але не всі сервіси так роблять. ( [джерело](https://datatracker.ietf.org/doc/html/rfc6376#section-8.6) )

Та інші методи з [RFC 6376](https://datatracker.ietf.org/doc/html/rfc6376#section-8).

### DMARC
- Атака шляхом експлуатації **RFC 5322** (поштовий заголовок складається з двох частин:  `display_name <hidden_email>`), через що можна абсолютно валідним чином відправляти розсилку, просто сподіваючись, що жертва не подивиться на адресу відправника.
- Атака на той же  **RFC 5322**, що допускає множинне введення адрес (`From: Vadim <user1@legit.com, user2@scam.com, user3@attack.org>`) таким чином, парсер серверу мусить обрати один для валідації. Відправивши декілька подібних листів можна спробувати зрозуміти який дійшов на особисту скриньку та почати більш масову розсилку. Якщо ж жодна конфігурація не дійшла є імовірність, що сервіс перевіряє їх усі, відтак можна відправити лист із сотнями запитів, сподіваючись, що у сервіса вийде час на перевірку листа і він його пропустить (або ж DDoS).
- Альтернативно, можна використати декілька заголовків `From` для того, аби парсер DMARC валідував за одним єдиним заголовком, в той час як MUA відображатиме інший. 
- Використання Leet Speak або схожих на оригінальні символи з тією ж метою введення жертви в оману (`i -> I ~ l <- L`). Скоріш за все, це буде виявлено антифродом SEG чи MUA фільтрами, але підхід вартий спроби.
- Відсутність `np` на кореневому домені, що дозволяє зловмиснику відправляти листи від неіснуючих сабдоменів, користуючись *Tree Walk* властивістю при пошуку `v=DMARC1` запису. Це може призвести як до валідації за пом'якшеними правилами кореневого домену (наприклад, `relaxed` замість `strict`, `p=none` і тп.), так і до відсутності DMARC перевірки, при відсутності DMARC запису до знаходження `psd=y`.
- М'яка політика `p=none`, що в цілому компрометує всю безпеку.
- Підміна DNS
- Тривале чи періодичне тестування із `t` чи `pct` параметрами, що дає достатній час для зловмисників на відправку пошти за процедурою пом'якшених параметрів захисту
- Атака на `rua` та `ruf` з метою приховання спроб обходу DMARC через DDoS, спроби XXE (спроби зчитування локальних файлів), Zip Bomb ( архів, що від мізерних ~ 64кб розпаковується до астрономічних значень на останньому шарі, займаючи всю пам'ять та апаратні ресурси ), XML Bomb (схожий принцип, але із XML сутностями). Відправлення фальшивих звітів для викривлення SIEM статистики.

Та інші методи з [RFC 9989](https://datatracker.ietf.org/doc/html/rfc9989#section-11).