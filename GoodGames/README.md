# GoodGames – Hack The Box (Retired)

## Overview

GoodGames is a Linux-based retired Hack The Box machine that demonstrates multiple
web application vulnerabilities, including SQL Injection and Server-Side Template Injection (SSTI).

This machine provides a realistic attack chain from initial access to remote code execution.

---

## Target Information

- IP Address: `10.10.11.130`
- Web Service: HTTP (Port 80)
- Web Server: Apache httpd 2.4.51
- Backend Framework: Werkzeug 2.0.2
- Language: Python 3.9.2
- Hostname: `goodgames.htb`

---

## Reconnaissance

Initial enumeration revealed a web application hosted on port 80.

![Dashboard Page](https://github.com/R-Galarza/htb-writeups-retired/blob/32c7c3cbf95b5bd40bbb73c1087cc8ecc1455fa5/GoodGames/images/dashboard.png)

The application presents a video game store with limited functionality
and a login form that enforces email validation on the client side.

![Login Page](https://github.com/R-Galarza/htb-writeups-retired/blob/1508dde9cd9d29c248644d5b3dcf2a704f6751d1/GoodGames/images/login.png)

---

## Vulnerability 1: SQL Injection (Authentication Bypass)

### Description

The login form restricts input to valid email formats, but this validation
is only enforced on the client side.

By intercepting the login request with Burp Suite, it is possible to inject
SQL payloads directly into the backend query.

### Proof of Concept

Authentication bypass via SQL Injection allowed access as a valid user.

This confirms that user input is not properly sanitized on the server side.

![SQLi Page](https://github.com/R-Galarza/htb-writeups-retired/blob/95a3d5e5b697c4b28cd8f426f67c8530c679a38b/GoodGames/images/sqli-web.png)

![SQLi2 Page](https://github.com/R-Galarza/htb-writeups-retired/blob/b585d158741ab82439cc5a8407a312243093162b/GoodGames/images/sqli-burp.png)

![SQLi3 Page](https://github.com/R-Galarza/htb-writeups-retired/blob/e377dc6cad4d6e6b6dd30b2fe4e61b0df749cf67/GoodGames/images/sqli-login-admin.png)

In order to retrieve additional data and expand the attack surface, an automated extraction was performed using **sqlmap**, based on a previously intercepted HTTP request.

The analysis successfully dumped sensitive information from the database, including valid credentials (which will not be disclosed in this writeup for ethical reasons).

However, the extracted credentials could not be directly reused, as there was no functional entry point within the application where they could be authenticated.

### Command Used

```bash
sqlmap -r request.txt --dump
```

### Impact

- Authentication bypass
- Unauthorized access to application features
- Exposure of backend database structure

---

## Subdomain Discovery

### Description

After successfully logging in as admin, I found an internal admin panel subdomain when I clicked the top right button::

```
http://internal-administration.goodgames.htb/login
````
![subdomain](https://github.com/R-Galarza/htb-writeups-retired/blob/b6202133abc8cd7c579f05f1f7f0a9ee31323d6a/GoodGames/images/Internal-administration-login.png)

The administrator credentials obtained during the **SQL Injection** They were reused in this subdomain.

![login](https://github.com/R-Galarza/htb-writeups-retired/blob/71e0b6ff4fc9629ad70f644cf9812b77d12feb33/GoodGames/images/login-internal.png)

![dash](https://github.com/R-Galarza/htb-writeups-retired/blob/6387912f840b0e79653d8eb5c9768b3ff263eca5/GoodGames/images/Internal-administration-dashboard.png)
### Impact

The exposure of an internal administrative interface to external users significantly increases the attack surface of the application.

By chaining this exposure with previously obtained administrator credentials, an attacker can gain full control over internal management functionality that was never intended to be publicly accessible.

This can lead to:

- Complete compromise of administrative features and sensitive backend operations  
- Unauthorized management of users, content, and application configuration  
- Increased risk of further lateral movement within internal services  
- Bypass of network-level trust boundaries and internal security assumptions  
- Full application takeover when combined with other vulnerabilities
---

## Vulnerability 2: Server-Side Template Injection (SSTI)

### Description

The profile editing functionality reflects user-controlled input
without proper sanitization.

![SSTI](https://github.com/R-Galarza/htb-writeups-retired/blob/f42e06812e1b05b15da175aab10f35d41bb14b54/GoodGames/images/SSTI.png)

Testing with template expressions confirmed the presence of SSTI.

![SSTI2](https://github.com/R-Galarza/htb-writeups-retired/blob/f42e06812e1b05b15da175aab10f35d41bb14b54/GoodGames/images/SSTI2.png)

### Proof of Concept

Payload used (Jinja2):

```jinja2
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
````

![SSTI-ID](https://github.com/R-Galarza/htb-writeups-retired/blob/436d73b67f5ff11bb943d8adff9a7789f569f7b0/GoodGames/images/SSTI-ID.png)

Successful execution confirms remote command execution.

### Impact

* Arbitrary command execution
* Full compromise of the application
* Access to sensitive files

---

## Flag Retrieval

Using SSTI RCE, the user flag was retrieved:

```jinja2
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat /home/augustus/user.txt').read() }}
```

---

## Remediation Recommendations

* Enforce server-side input validation
* Implement proper authorization checks on all endpoints
* Sanitize template inputs
* Apply the principle of least privilege
* Avoid exposing internal subdomains

---

## Disclaimer

This writeup is based on a retired Hack The Box machine
and is intended for educational purposes only.

