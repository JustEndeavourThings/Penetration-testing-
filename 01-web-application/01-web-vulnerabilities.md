# Web Application Vulnerabilities

> Mirrored from Notion → **Web application → Nothing Much**
> Source: `Nothing Much, Just Endeavour's things` database

A running index of web application vulnerability classes, how each one works,
how to test for it, and the lab/payload steps that go with it.

## Contents

**Exported**

1. [File upload vulnerability](#1-file-upload-vulnerability)
2. [Server-side request forgery (SSRF)](#2-server-side-request-forgery-ssrf)
3. [Authentication vulnerabilities](#3-authentication-vulnerabilities)
4. [Business logic vulnerabilities](#4-business-logic-vulnerabilities)
5. [Prototype pollution](#5-prototype-pollution)
6. [Information disclosure](#6-information-disclosure)
7. [Race conditions](#7-race-conditions)
8. [Host header injection](#8-host-header-injection)
9. [HTTP request smuggling](#9-http-request-smuggling)
10. [Web socket vulnerabilities](#10-web-socket-vulnerabilities)
11. [Server-side template injection (SSTI)](#11-server-side-template-injection-ssti)
12. [JWT attacks](#12-jwt-attacks)

**Pending export** (present in Notion, body text not yet pulled)

13. Insecure deserialization
14. Web cache poisoning
15. Web cache deception
16. SQLMAP and SQLi
17. NoSQL injection
18. CRLF injection
19. Directory listing
20. Local file inclusion (LFI)
21. DDL hijacking
22. API testing
23. Gobuster

---

## 1. File upload vulnerability

A file upload vulnerability happens when a web application allows users to upload
files without proper validation, letting an attacker upload harmful files.

The impact of file upload vulnerabilities generally depends on two key factors:

- Which aspect of the file the website fails to validate properly — whether that
  be its size, type, contents, and so on.
- What restrictions are imposed on the file once it has been successfully uploaded.

Impact:

- A server-side code file that functions as a **web shell**, effectively granting
  full control over the server.
- If the filename isn't validated properly, an attacker could overwrite critical
  files simply by uploading a file with the same name. If the server is also
  vulnerable to directory traversal, attackers may even be able to upload files
  to unanticipated locations.
- Failing to make sure that the size of the file falls within expected thresholds
  could also enable a form of **denial-of-service (DoS)** attack, whereby the
  attacker fills the available disk space.

### Encoding the upload

`application/x-www-form-url-encoded` is fine for sending simple text like your
name or address. However, it isn't suitable for sending large amounts of binary
data, such as an entire image file or a PDF document. In this case, the content
type `multipart/form-data` is preferred.

### Exploitation techniques

1. **Exploiting unrestricted file uploads to deploy a web shell**
   ```php
   <?php echo file_get_contents('/path/to/target/file'); ?>
   ```
2. **Web shell upload via Content-Type restriction bypass**
   Just change the content type to `image/jpeg` after intercepting.
3. **Web shell upload via path traversal**
   ```
   Content-Disposition: form-data; name="avatar"; filename="../exploit.php"
   ```

### Prevention

- Check the file extension against a whitelist of permitted extensions rather
  than a blacklist of prohibited ones.
- Make sure the filename doesn't contain any substrings that may be interpreted
  as a directory traversal sequence (e.g. `../`).
- Rename uploaded files to avoid collisions that may cause existing files to be
  overwritten.
- Do not upload files to the server's permanent filesystem until they have been
  fully validated.
- As much as possible, use an established framework for preprocessing file
  uploads rather than attempting to write your own validation mechanisms.

---

## 2. Server-side request forgery (SSRF)

Useful listener: `webhook.site`

Server-side request forgery is a web security vulnerability that allows an
attacker to cause the server-side application to make requests to an unintended
location.

In a typical SSRF attack, the attacker might cause the server to make a
connection to internal-only services within the organization's infrastructure.
In other cases, they may be able to force the server to connect to arbitrary
external systems. This could leak sensitive data, such as authorization
credentials.

SSRF attacks often exploit trust relationships to escalate an attack from the
vulnerable application and perform unauthorized actions. These trust
relationships might exist in relation to the server, or in relation to other
back-end systems within the same organization.

### Testing

1. **Basic SSRF against the local server**
   Change the URL in the `stockApi` parameter to `http://127.0.0.1/admin`. This
   should display the administration interface. Then:
   `http://localhost/admin/delete?username=carlos`
2. **Basic SSRF against another back-end system**
   Same as before — send the request to Intruder with
   `stockapi=http://192.168.0.1:8080/admin`, then run a sniper attack with the
   numbered list from 1 to 254 on the class D of the IP address. The request with
   `200 OK` gives the live host — take that URL and delete carlos.

3. **SSRF with blacklist-based input filter**
   To bypass the anti-SSRF security, try alternate representations of the same
   address (e.g. `127.1`, `2130706433`, `0x7f000001`, `127.0.0.1.nip.io`) and
   double-URL-encode the blocked characters.
4. **SSRF with filter bypass via open redirection**
   Here there can be a redirect in the URL in the `stockApi` parameter:
   ```
   /product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=wiener
   ```

### Blind SSRF

Blind SSRF vulnerabilities arise when an application can be induced to issue a
back-end HTTP request to a supplied URL, but the response from the back-end
request is not returned in the application's front-end response.

- **Blind SSRF with out-of-band detection** — paste the Burp Collaborator ID
  (`http://<collaborator-id>`) into the `Referer` header.
- **Blind SSRF with Shellshock exploitation** — *requires revision.*

### Extra SSRF notes

Start a logging server and capture everything the target sends back:

```bash
sudo chmod +x server.py && sudo python3 server.py
```

This logs the data into `data.html`. Open the `data.html` file — it contains all
the essential information related to the server that can be used to launch
further attacks.

Example targets / lab URLs:

```
http://hrms.thm/?url=localhost/url
http://hrms.thm/url.php?id=192.168.2.10/bigImage.jpg
```

> SSRF lab in TryHackMe.

---

## 3. Authentication vulnerabilities

There are three main types of authentication:

- Something you **know**, such as a password or the answer to a security
  question. These are sometimes called *"knowledge factors"*.
- Something you **have** — a physical object such as a mobile phone or security
  token. These are sometimes called *"possession factors"*.
- Something you **are** or do — for example your biometrics or patterns of
  behaviour. These are sometimes called *"inherence factors"*.

Most vulnerabilities in authentication mechanisms occur in one of two ways:

- The authentication mechanisms are **weak** because they fail to adequately
  protect against brute-force attacks.
- **Logic flaws or poor coding** in the implementation allow the authentication
  mechanisms to be bypassed entirely by an attacker. This is sometimes called
  *"broken authentication"*.

### Username enumeration

1. **Via different responses** — use Burp Intruder and check for length changes;
   the username and password lists are given in the labs themselves.
2. **Via subtly different responses** — in Intruder, under *Settings → Grep –
   Extract*, add `Invalid username and password.` and start the attack. You will
   see the difference; take the username and then do the same for the password.
3. **Via response timing** — *need to study this.*
   Reference: [Authentication Vulnerabilities – Lab #5 Username enumeration via
   response timing](https://infosecwriteups.com/authentication-vulnerabilities-lab-5-username-enumeration-via-response-timing-16b07552275a)

### Brute-force protection flaws

1. **Broken brute-force protection, IP block** — there is a flaw in the app
   where the max attempts is 2, but it can be reset by logging in with correct
   credentials. Create a payload of usernames and passwords such that you
   alternate a correct login with brute-force attempts. Also, in the Intruder
   *Resource pool*, set max concurrent requests to `1` and create a new resource
   pool.
2. **Username enumeration via account lock** — when you try a correct username
   with a wrong password it shows "too many attempts". Add the payload to both
   username and password such that `=peter$$`, paste all of the username list
   into payload one and start the attack. The request with the long length is
   the valid user (confirmed with "too many attempts"). Then put that username in
   the username parameter, paste the password list into the password parameter
   and start the attack — the request with the big length is the password.
3. **Broken brute-force protection, multiple credentials per request** — the
   login request is sent in JSON format. Supply multiple passwords as an array:
   ```json
   { "username": "carlos", "password": ["passwd1", "passwd2"] }
   ```
   Send it and the response will be `302 Found`.

### HTTP basic authentication

In HTTP basic authentication, the client receives an authentication token from
the server, which is constructed by concatenating the username and password and
encoding it in Base64. This token is stored and managed by the browser, which
automatically adds it to the `Authorization` header of every subsequent request:

```
Authorization: Basic base64(username:password)
```

### Multi-factor authentication (MFA)

1. **2FA simple bypass** — the user is effectively in a "logged in" state before
   they have entered the verification code, so they can access the other pages
   that were only meant to be available after login.
2. **2FA broken logic** — a `GET /login2` request is used to send the OTP to the
   user and `POST /login2` verifies it. Intercept the GET request and change the
   `verify` parameter, then brute-force the POST request (changing the `verify`
   parameter there as well).
3. **2FA bypass using a brute-force attack** — *need to do this using a macro.*

### Vulnerabilities in other authentication mechanisms

**Brute-forcing a stay-logged-in cookie** — when you log in with "remember me"
ticked, the browser creates a `stay-logged-in` cookie in the form:

```
base64(username + ':' + md5HashOfPassword)
# e.g. wiener:51dc30ddc473d43a6011e9ebba6ca770
```

Send a `GET /my-account` request, change the username to `carlos`, delete the
current `sessionid` so only `stay-logged-in` remains, send it to Intruder and add
a payload. In the payload config add the `Hash: MD5` processing with the prefix
`carlos:` and then Base64-encode it, add the password list and start the attack.

### OAuth

For any OAuth grant type, the client application has to specify which data it
wants to access and what kind of operations it wants to perform. It does this
using the `scope` parameter of the authorization request it sends to the OAuth
service.

**Authorization code grant type**

1. Authorization request
2. User login and consent
3. Authorization code grant
4. Access token request
5. Access token grant
6. API call
7. Resource grant

**Implicit grant type**

1. Authorization request
2. User login and consent
3. Access token grant
4. API call
5. Resource grant

---

## 4. Business logic vulnerabilities

Business logic vulnerabilities are flaws in the design and implementation of an
application that allow an attacker to elicit unintended behaviour. This
potentially enables attackers to manipulate legitimate functionality to achieve a
malicious goal. These flaws are generally the result of failing to anticipate
unusual application states that may occur and, consequently, failing to handle
them safely.

1. **Excessive trust in client-side controls** — intercept the request that adds
   the item to the cart and change the price.
2. **2FA broken logic** — OTP bypassing. *Need to perform the lab.*
3. **High-level logic vulnerability** — the app lets us put a negative quantity
   in the cart through interception. Add two items, one negative and one
   positive, and you can buy the jacket for a much smaller amount.
4. **Low-level logic flaw** — we can put 99 quantity of an item at once through
   Repeater. First use a `NULL` payload with "continue indefinitely" to add items
   an unlimited number of times, so the buffer overflows and the total price goes
   negative. Then buy another item and — with some calculation — call it enough
   times through Intruder (with generated payload options) to bring the price back
   from negative to positive and below 100.
5. **Inconsistent handling of exceptional input** — see the lab write-up below.
6. **Inconsistent security controls** — register an account with an email we
   control, then use the update-email option to change it to
   `@dontwannacry.com` (a domain referenced by the application itself). Once the
   email is changed, the admin interface becomes available.
7. **Weak isolation on dual-use endpoint** — log in and update the password;
   intercept that request, change the username to `administrator` and remove the
   current-password parameter. You can now change the administrator's password
   and log in as admin.
8. **Password reset broken logic** — intercept the forgotten-password request and,
   after clicking the link in the email, intercept that request and change
   `wiener` to `carlos` — you have now changed carlos's password.
9. **2FA simple bypass** — there is no restriction when going from step 1 to step
   3. Log in with the `wiener` account to see the URL of the third step, then log
   in as `carlos` and, on the email verification step, change the URL from step 2
   to step 3, thereby bypassing email verification.
10. **Insufficient workflow validation** — buy an item first to see steps 2 and 3,
    then add the item to the cart and change the URL from the cart to the
    already-purchased URL you saw earlier.

11. **Authentication bypass via flawed state machine** — log in with `wiener`; the
    application then asks for the type of user. Intercept that request and drop
    it, and you are logged in as admin.
12. **Flawed enforcement of business rules** — when one coupon is applied another
    coupon can be applied after it, and multiple coupons can be applied multiple
    times, alternating first this then that.
13. **Infinite money logic flaw**
14. **Authentication bypass via encryption oracle**
15. **Bypassing access controls using email address parsing discrepancies**
---

## 5. Prototype pollution

Prototype pollution is a JavaScript vulnerability that enables an attacker to add
arbitrary properties to global object prototypes, which may then be inherited by
user-defined objects.

- **Source** — any JavaScript property or function that accepts data that can be
  controlled by an external party, such as a user or an attacker. Example:
  `document.cookie` (data stored in cookies).
- **Sink** — a potentially dangerous JavaScript function or DOM object that can
  process or execute data passed to it. Example: `alert()` generates an alert in
  the HTML page.
- **Gadget** — code such as `alert(obj.message);` that uses a property;
  a gadget is code that passes user-controlled data into that sink.
- **Prototype pollution source** — a special type of source where the attacker can
  inject a property that modifies the JavaScript prototype:
  ```json
  { "__proto__": { "admin": true } }
  ```
- **Prototype pollution sink** — a JavaScript function or DOM element that you're
  able to access via prototype pollution, which enables you to execute arbitrary
  JavaScript or system commands.
- **Prototype pollution gadget** — the chain that turns a pollution source into a
  usable sink.

---

## 6. Information disclosure

Information disclosure, also known as information leakage, is when a website
unintentionally reveals sensitive information to its users.

Reasons:

- **Failure to remove internal content from public content.**
- **Insecure configuration of the website and related technologies.**
- **Flawed design and behavior of the application.**

Testing techniques:

1. **Information disclosure in error messages** — `productid=1` vs
   `productid=abcd`; the error reveals the server version.
2. **Information disclosure on debug page** — discover content on the target, find
   the `cgi-bin` page and read the `secret-key` in it.
3. **Source code disclosure via backup files** — same as before, or through
   `robots.txt`.
4. **Authentication bypass via information disclosure** — `TRACE` is accepted and
   reveals that the `X-IP-Authorisation` header is what the server uses to
   authorise users. Add that header to the request with the loopback address and
   you can reach the admin panel. In Burp, use *Proxy → Match and Replace* with
   "request header" as the type and `X-IP-Authorisation: 127.0.0.1` as the
   replacement.
5. **Information disclosure in version control history** — access the `/.git`
   directory and download it:
   ```bash
   wget -r https://YOUR-LAB-ID.web-security-academy.net/.git/
   ```
   Then use `git log -p` in the downloaded directory to read the commit history —
   the admin password shows up in the config history, and you can log in with it.

---

## 7. Race conditions

Race conditions are a common type of vulnerability closely related to business
logic flaws. They occur when websites process requests concurrently without
adequate safeguards. This can lead to multiple distinct threads interacting with
the same data at the same time, resulting in a "collision" that causes unintended
behaviour in the application. The period of time during which a collision is
possible is known as the **"race window"**.

Limit overruns are a subtype of so-called "time-of-check to time-of-use"
(TOCTOU) flaws.

**Network jitter** refers to the variation in packet arrival time during data
transmission over a network. Ideally data packets should arrive at consistent
intervals — but in real networks delays vary due to congestion, hardware issues or
routing changes. This inconsistent delay is called **jitter**.

**Limit overrun race conditions** — where there is a coupon-code redemption
option, send it to Repeater, open the *Custom actions* panel, choose *New → from
template → Trigger a race condition*, and run the request 20 times in parallel.
Then remove the coupon on the web UI, run the attack again, refresh, and buy the
jacket.

---

## 8. Host header injection

Host Header Injection is a web vulnerability that occurs when an application
improperly trusts or reflects the `Host` header from an HTTP request without
validating or sanitising it.

> Some intercepting proxies derive the target IP address from the Host header
> directly, which makes this kind of testing all but impossible; any change you
> make to the header just causes the request to be sent to a completely different
> IP address.

Impact of host header injection:

1. Web cache poisoning
2. Password reset poisoning
3. Virtual host routing abuse
4. Bypass security controls
5. Open redirect / content injection
6. Business logic flaws in specific functionality
7. Classic server-side vulnerabilities, such as SQL injection

Test payload:

```
Host: vulnerable-website.com:bad-stuff-here
```

---

## 9. HTTP request smuggling

HTTP request smuggling is a technique for interfering with the way a web site
processes sequences of HTTP requests that are received from one or more users.

| Type | Front-end uses | Back-end uses |
|---|---|---|
| **CL.TE** | `Content-Length` | `Transfer-Encoding` |
| **TE.CL** | `Transfer-Encoding` | `Content-Length` |
| **TE.TE** | `Transfer-Encoding` | `Transfer-Encoding` (one server is induced not to process it by obfuscating the header) |

Labs / techniques:

1. **Basic CL.TE vulnerability** — manipulate the server into believing the
   current request is complete and a new one has started. Change HTTP/2 to HTTP/1,
   change the request method and everything else, and paste the smuggling payload.
2. **Basic TE.CL vulnerability** — same as above.
3. **Obfuscating the TE header** — same as before.
4. **Confirming a CL.TE vulnerability via differential responses**
5. **Confirming a TE.CL vulnerability via differential responses**
6. **Bypassing front-end security controls (CL.TE)**
7. **Bypassing front-end security controls (TE.CL)**
8. **Revealing front-end request rewriting** — when we use any feature of the web
   application the front-end creates its own headers; use the search feature and
   the response gives the IP address of the current user and the unique header
   that can be used to bypass into the admin account.

---

## 10. Web socket vulnerabilities

1. **Manipulating WebSocket messages to exploit vulnerabilities** — WebSockets are
   often used for chatting and streaming. We can see the history in Burp Suite;
   intercept the message, manipulate the request and exploit the vulnerability by
   injecting a payload. Intercept the request and put:
   ```html
   <img src=1 onerror='alert(1)'>
   ```
   and it will generate the alert.
2. **Manipulating the WebSocket handshake to exploit vulnerabilities** — same as
   before, but the payload got our IP address blocked, so we used
   `X-Forwarded-For: 1.1.1.1` to spoof the IP address and then sent:
   ```html
   <img src=1 oNeRrOr=alert1>
   ```
   which is accepted and generates the alert.
3. **Cross-site WebSocket hijacking** — paste the exploit in the exploit server
   along with the WebSocket handshake URL and the Collaborator URL as the payload,
   then deliver the payload to the victim. Everything the victim does (changing a
   password, requesting a password, etc.) is logged in Collaborator.

**Mitigation:** use `wss://` (encryption) instead of `ws://`, protect against CSRF,
and handle data correctly on both the server and client side.

---

---

## 11. Server-side template injection (SSTI)

Templates allow websites to automatically fill data into pages instead of creating
separate pages for everything.

Server-side template injection is when an attacker is able to use native template
syntax to inject a malicious payload into a template, which is then executed
server-side.

Sometimes we come across a website that displays info like "hello Zeeshan" — this
uses template functionality, so if Ali logs in it shows "hello Ali". If the
template sanitises correctly then it's fine; but if the web application uses the
data directly and does not sanitise it, we can add a payload and it will be used
as code and executed.

### Methodology

1. **Detect** — plain context, code context.
2. **Identify** — fingerprint the template engine (a decision chart helps here).
3. **Exploit**

Detection payloads:

```
{{7*7}}   ${7*7}   #{7*7}   <%= 7*7 %>
```

### Labs

1. **Basic server-side template injection** — clicking "view details" shows
   "unfortunately..." in the URL itself. Add `<%= 7*7 %>`; if "unfortunately"
   becomes `49` the parameter is vulnerable. Change the payload to:
   ```
   <%= system("id") %>
   ```
   in the URL or in the intercepted request, and use it to delete `morale.txt`.
2. **Basic server-side template injection (code context)** — after login there is a
   "preferred name" feature that uses `user.name`. Search for the reflection: when
   you comment, the username is reflected. Intercept the preferred-name request and
   add the payload:
   ```
   }}{% import os %} {{os.system('pwd')}}
   ```
   The leading `}}` escapes the previous template. When the payload produced an
   error saying the application didn't recognise the syntax, it revealed that the
   **Tornado** template engine is in use.

---

## 12. JWT attacks

JSON Web Tokens (JWTs) are a standardised format for sending cryptographically
signed JSON data between systems. Unlike classic session tokens, all the data the
server needs is stored client-side within the JWT itself. A JWT consists of three
parts: a **header**, a **payload** and a **signature**.

With a symmetric algorithm the signature is made by base64url-encoding the header
and payload and then:

```
data = header_b64 + "." + payload_b64
signature = HMAC-SHA256(data, secret_key)
```

The server will reject a token with no signature.

Asymmetric tokens use a key pair — `public` (`n`, `e`) and `private`
(`d`, `p`, `q`, `dp`, `dq`, `qi`) components:

- The auth server (Okta, Auth0, Keycloak, or your own) generates the pair.
- The **private key** is kept secret inside the auth server.

**Impact:** escalate your own privileges or impersonate other users, taking full
control of their accounts.

**How it arises:** flawed JWT handling within the application itself.

### Labs

1. **JWT authentication bypass via unverified signature** — the developer only
   calls `decode()` and never `verify()`. The token is
   `header.payload.signature`. Capture the request; `/admin` is only accessible to
   `administrator`. Select only the payload section of the JWT, change `wiener` to
   `administrator`, apply the changes and delete carlos.
2. **JWT authentication bypass via `alg: none`** — the token is accepted without a
   signature. Decode the header, set the algorithm to `none`, delete the signature
   but leave the trailing dot in place, then change the username in the payload to
   `administrator`.

---

> **Import status:** sections 1–12 above are mirrored from Notion. The topics
> listed below exist in the Notion page but their body text has not been exported
> yet:
>
> 13. Insecure deserialization
> 14. Web cache poisoning
> 15. Web cache deception
> 16. SQLMAP and SQLi
> 17. NoSQL injection
> 18. CRLF injection
> 19. Directory listing
> 20. Local file inclusion (LFI)
> 21. DDL hijacking
> 22. API testing
> 23. Gobuster
