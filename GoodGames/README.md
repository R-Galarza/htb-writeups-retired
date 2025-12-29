# GoodGames – Hack The Box (Retired)

## Overview

GoodGames is a Linux-based retired Hack The Box machine that demonstrates multiple
web application vulnerabilities, including SQL Injection, Broken Access Control,
and Server-Side Template Injection (SSTI).

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

The application presents a video game store with limited functionality
and a login form that enforces email validation on the client side.

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

### Impact

- Authentication bypass
- Unauthorized access to application features
- Exposure of backend database structure

---

## Vulnerability 2: Broken Access Control

### Description

After authentication, a settings icon redirects to an internal administration panel:

```
http://internal-administration.goodgames.htb/login

````

This subdomain is not protected by proper authorization checks.

Once added to the hosts file, the admin panel becomes accessible to unauthorized users.

### Impact

- Access to administrative functionality
- Privilege escalation
- Violation of access control boundaries

---

## Vulnerability 3: Server-Side Template Injection (SSTI)

### Description

The profile editing functionality reflects user-controlled input
without proper sanitization.

Testing with template expressions confirmed the presence of SSTI.

### Proof of Concept

Payload used (Jinja2):

```jinja2
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
````

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

## OWASP Top 10 Mapping

* A01: Broken Access Control
* A03: Injection
* A07: Identification & Authentication Failures

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

````
