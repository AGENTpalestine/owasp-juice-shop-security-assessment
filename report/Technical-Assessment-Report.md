# OWASP Juice Shop — Web Application Security Assessment

**Assessment Type:** Web Application Security Assessment
**Target:** OWASP Juice Shop
**Application Version:** 20.3.0
**Assessment Period:** November 9–10, 2026
**Deployment:** Docker
**Operating System:** Kali Linux
**Container:** juice-shop
**Target URL:** `http://127.0.0.1:3000`
**Environment:** Local Educational Lab
**Assessment Status:** Testing Completed — Findings Documented

---

## 1. Executive Summary

This report documents a security assessment of OWASP Juice Shop, a deliberately vulnerable web application deployed in a local Docker-based lab environment.

The assessment included deployment verification, service discovery, web technology identification, HTTP response inspection, security header review, automated reconnaissance, and manual verification of potentially exposed resources.

The investigation confirmed two related exposure conditions:

1. A publicly accessible web directory listing that disclosed resource names.
2. Unauthenticated access to `acquisitions.md`, a document explicitly marked as confidential.

Additional tests examined other resources, HTTP security headers, and selected backup-style files. Some automated scanner observations were determined to be false positives after manual verification.

The assessment was limited to the endpoints and resources documented in this report. It does not represent a comprehensive penetration test of every Juice Shop feature or API endpoint.

## 2. Assessment Objectives

The objectives were to:

- Verify application deployment and availability.
- Identify exposed network services.
- Inspect web application technologies and HTTP responses.
- Identify publicly accessible directories and files.
- Verify potentially sensitive resources manually.
- Review selected HTTP security headers.
- Distinguish confirmed findings from scanner false positives.
- Document evidence and recommend remediation.

## 3. Scope and Limitations

### 3.1 In-Scope

The assessment covered:

- The local OWASP Juice Shop web application.
- TCP port 3000 on `127.0.0.1`.
- The application-accessible `/ftp/` directory.
- Selected files discovered through the directory listing.
- HTTP response headers from tested endpoints.
- Docker container status and application availability.

### 3.2 Out-of-Scope

The following activities were not performed:

- Testing external systems or third-party infrastructure.
- Denial-of-Service testing.
- Attacks against real users or real accounts.
- Source code review.
- Comprehensive testing of all application APIs.
- Full infrastructure penetration testing.

### 3.3 Limitations

The assessment covered only the resources and endpoints explicitly tested.

Automated scanner results were treated as preliminary observations and were manually verified where possible.

The results do not establish the security status of every component, authentication workflow, or business logic function in the application.

The application is deliberately vulnerable and was tested in a local educational environment. Findings should not be interpreted as evidence about a production deployment.

## 4. Assessment Environment

### 4.1 Infrastructure

| Component | Configuration |
|---|---|
| Operating System | Kali Linux |
| Container Engine | Docker 28.5.2 |
| Application | OWASP Juice Shop |
| Application Version | 20.3.0 |
| Docker Image | bkimminich/juice-shop:latest |
| Runtime | Node.js v24.21.0 |
| Container Name | juice-shop |
| Internal Port | TCP 3000 |
| Target Address | 127.0.0.1:3000 |

### 4.2 Deployment Configuration

The application was deployed using Docker with the following configuration:

`docker run -d --name juice-shop --restart unless-stopped -p 127.0.0.1:3000:3000 bkimminich/juice-shop:latest`

The port binding restricts access through the host's loopback interface rather than exposing the application directly on all network interfaces.

### 4.3 Application Availability

**Supporting Evidence:**

- `evidence/raw/01-docker-status.txt`
- `evidence/raw/02-docker-logs.txt`

The application was verified as running in its Docker container.

Observed results:

- Container status: Running.
- Published port: `127.0.0.1:3000`.
- HTTP response: `200 OK`.
- Content type: `text/html`.
- Page title: `OWASP Juice Shop`.
- Application startup log: `Server listening on port 3000`.

These observations confirmed that the application was available for testing.

## 5. Assessment Methodology

The assessment followed these stages:

1. **Deployment verification:** Inspect Docker status, port mapping, and application logs.
2. **Service discovery:** Identify the exposed TCP service.
3. **Technology identification:** Inspect the application response and visible technologies.
4. **Automated reconnaissance:** Use web scanning tools to identify potential resources and configuration issues.
5. **Manual verification:** Test relevant paths and inspect actual HTTP responses.
6. **Security review:** Evaluate confirmed exposure conditions and selected HTTP headers.
7. **Documentation:** Record findings, evidence, limitations, and remediation recommendations.

Automated tool output was not considered sufficient proof of a vulnerability without supporting verification.

## 6. Tools Used

| Tool | Purpose |
|---|---|
| Docker | Container deployment and inspection |
| Nmap | TCP service discovery and service detection |
| WhatWeb | Web technology identification |
| Nikto | Automated web reconnaissance |
| cURL | HTTP requests and response inspection |
| Linux utilities | Evidence management and analysis |
| Web browser | Manual inspection and screenshots |

## 7. Reconnaissance and Enumeration

### 7.1 TCP Service Discovery — Nmap

Nmap was used to inspect the target TCP port and identify the exposed service.

Command executed:

`nmap -sV -sC -p 3000 127.0.0.1`

Observed results:

- TCP port `3000` was open.
- The service returned an HTTP response.
- The application returned HTTP `200 OK`.
- An HTTP OPTIONS request returned status `204`.
- The observed CORS methods included GET, HEAD, PUT, PATCH, POST, and DELETE.
- Nmap did not confidently identify the underlying service version.

The presence of HTTP methods in an OPTIONS response does not independently demonstrate that unauthorized operations are possible.

**Evidence:** `evidence/raw/03-nmap.txt`

### 7.2 Web Technology Identification — WhatWeb

WhatWeb was used to identify visible web technologies and response characteristics.

Observed results:

- HTTP status: `200 OK`.
- Page title: `OWASP Juice Shop`.
- HTML5 was detected.
- `X-Frame-Options: SAMEORIGIN` was present.
- `X-Content-Type-Options: nosniff` was present.
- `Access-Control-Allow-Origin: *` was observed.
- No definitive server version was identified.

The wildcard CORS header requires endpoint-specific investigation before it can be classified as a security vulnerability.

**Evidence:** `evidence/raw/04-whatweb.txt`

### 7.3 HTTP Response Inspection

The root application endpoint was inspected using cURL.

Observed results:

| Attribute | Result |
|---|---|
| Target | 127.0.0.1:3000 |
| HTTP Status | 200 OK |
| Content Type | text/html |
| Page Title | OWASP Juice Shop |

These results confirmed that the application was reachable during the assessment.

**Evidence:**

- `evidence/raw/06-http-response.txt`
- `evidence/raw/06-page-title.txt`

### 7.4 Automated Web Reconnaissance — Nikto

Nikto was used to identify potentially exposed resources and common web configuration issues.

The scan generated multiple observations, including references to potentially sensitive filenames and directories.

These observations were treated as preliminary leads. Each relevant path required manual verification to determine whether it exposed an actual resource or returned a normal application response.

**Evidence:** `evidence/raw/07-nikto.txt`

### 7.5 Manual Verification of Discovered Paths

Selected paths identified during automated reconnaissance were tested manually.

| Path | HTTP Status | Observation |
|---|---:|---|
| `/robots.txt` | 200 | Robots exclusion file returned |
| `/ftp/` | 200 | Directory listing accessible |
| `/.htpasswd` | 200 | SPA fallback response characteristics |
| `/.bash_history` | 200 | SPA fallback response characteristics |
| `/userdata.json` | 200 | SPA fallback response characteristics |
| `/login.json` | 200 | SPA fallback response characteristics |

Several unrelated paths returned responses with the same size, approximately 9984 bytes. This behavior was consistent with the application's single-page application fallback rather than confirmed disclosure of the requested files.

These paths were not classified as confirmed file exposure findings.

**Evidence:** `evidence/raw/08-nikto-manual-verification.txt`

### 7.6 Robots.txt Inspection

The `/robots.txt` resource returned HTTP `200 OK`.

Its observed content was:

`User-agent: *`
`Disallow: /ftp`

This directive requests compliant search engine crawlers not to access the specified path.

It does not enforce access control and does not prevent users from requesting the path directly.

**Evidence:** `evidence/raw/09-robots-body.txt`

## 8. Web Directory and Resource Exposure

### 8.1 Web Directory Listing at /ftp/

The following endpoint was accessible:

`http://127.0.0.1:3000/ftp/`

The server returned HTTP `200 OK` and an HTML directory listing.

The listing exposed references to multiple resources, including:

- `quarantine`
- `acquisitions.md`
- `announcement_encrypted.md`
- `coupons_2013.md.bak`
- `eastere.gg`
- `encrypt.pyc`
- `incident-support.kdbx`
- `legal.md`
- `package-lock.json.bak`
- `package.json.bak`
- `suspicious_errors.yml`

The directory listing made these resource names discoverable without authentication.

The captured HTML response was incomplete according to cURL, which reported a transfer that ended 59 bytes before the expected response length. Nevertheless, the captured content included the directory listing title and visible resource links. The response headers also reported HTTP `200 OK` and a content length of 11274 bytes.

**Evidence:**

- `evidence/raw/10-ftp-headers.txt`
- `evidence/raw/10-ftp-response.html`
- `evidence/raw/11-ftp-verification-headers.txt`
- `evidence/screenshots/01-ftp-directory-listing.png`

### 8.2 Unauthenticated Access to acquisitions.md

The following resource was requested directly:

`/ftp/acquisitions.md`

Observed response:

| Attribute | Result |
|---|---|
| HTTP Status | 200 OK |
| Content Type | text/markdown; charset=UTF-8 |
| Response Size | 909 bytes |
| Authentication Required | No authentication observed |

The document contained the heading:

`# Planned Acquisitions`

It also included the statement:

`This document is confidential! Do not distribute!`

The successful unauthenticated response confirms that a document explicitly marked as confidential was accessible through the tested endpoint.

The actual business sensitivity and authenticity of the document were not independently established.

**Evidence:**

- `evidence/raw/12-acquisitions-response.txt`
- `evidence/raw/12-ftp-file-headers.txt`
- `evidence/screenshots/02-sensitive-file-exposure.png`

### 8.3 announcement_encrypted.md

The resource `/ftp/announcement_encrypted.md` returned HTTP `200 OK`.

The captured response contained approximately 369 KB of numeric data.

The observed data was not human-readable in its captured form. The available evidence does not establish the encoding, encryption algorithm, key, or original message content.

No claim about successful decryption or cryptographic weakness is made.

**Evidence:**

- `evidence/raw/15-announcement-headers.txt`
- `evidence/raw/15-announcement-response.txt`

### 8.4 legal.md

The resource `/ftp/legal.md` returned HTTP `200 OK`.

The response contained approximately 3047 bytes of Markdown content. The inspected text appeared to contain placeholder content resembling Lorem ipsum.

No sensitive information was confirmed in the inspected content.

**Evidence:**

- `evidence/raw/16-legal-headers.txt`
- `evidence/raw/16-legal-response.txt`

### 8.5 Backup-Style Resource Verification

Two backup-style resources were tested directly.

| Resource | HTTP Status | Result |
|---|---:|---|
| `/ftp/package.json.bak` | 403 | Access denied |
| `/ftp/coupons_2013.md.bak` | 403 | Access denied |

The returned HTML error response included the message:

`Only .md and .pdf files are allowed!`

These tests demonstrate that the two requested resources were denied under the tested conditions.

They are recorded as negative test results and are not classified as confirmed vulnerabilities.

**Evidence:**

- `evidence/raw/13-package-backup-headers.txt`
- `evidence/raw/13-package-backup-response.txt`
- `evidence/raw/14-coupons-bak-headers.txt`
- `evidence/raw/14-coupons-bak-denied-response.html`

## 9. HTTP Security Header Review

### 9.1 Observed Headers

The HTTP response headers were reviewed for selected browser security controls.

| Header | Observed Result | Assessment |
|---|---|---|
| `X-Frame-Options` | `SAMEORIGIN` | Present |
| `X-Content-Type-Options` | `nosniff` | Present |
| `Access-Control-Allow-Origin` | `*` | Requires endpoint-specific review |
| `Content-Security-Policy` | Not observed | Hardening opportunity |
| `Referrer-Policy` | Not observed | Hardening opportunity |
| `Permissions-Policy` | Not observed | Hardening opportunity |
| `Strict-Transport-Security` | Not observed | HTTPS deployment review required |

### 9.2 CORS Observation

The tested response included:

`Access-Control-Allow-Origin: *`

A wildcard origin on the inspected response does not independently establish an exploitable CORS vulnerability.

A complete CORS assessment would require testing relevant API endpoints, response behavior, allowed origins, and credential handling.

No confirmed CORS vulnerability is claimed in this report.

### 9.3 Missing Security Headers

The inspected response did not include Content-Security-Policy, Referrer-Policy, or Permissions-Policy.

Their absence is documented as a security-hardening opportunity, not as proof of an exploitable vulnerability.

Strict-Transport-Security is intended for HTTPS deployments. Its absence on the tested HTTP localhost endpoint is not classified as a confirmed vulnerability.

**Evidence:** `evidence/raw/05-security-headers.txt`

## 10. Security Findings

This section summarizes the security issues confirmed during manual verification.

### 10.1 F-01 — Unauthenticated Web Directory Listing

**Affected Endpoint:** `/ftp/`
**Finding Category:** Information Disclosure / Directory Listing
**Observed HTTP Status:** `200 OK`
**Verification Status:** Manually Verified

#### Description

The application exposed a web-accessible directory at /ftp/ through HTTP.

A request to `/ftp/` returned an HTML directory listing containing references to multiple files and resources.

The listing allowed unauthenticated visitors to discover resource names, including documents and backup-style files.

#### Evidence

- `evidence/raw/10-ftp-headers.txt`
- `evidence/raw/10-ftp-response.html`
- `evidence/screenshots/01-ftp-directory-listing.png`

#### Potential Security Impact

Public directory listings may disclose internal filenames, document names, and application resources.

This information can assist an attacker in identifying additional resources that may contain sensitive information or have insufficient access controls.

The actual impact depends on the sensitivity and accessibility of the listed resources.

#### Recommended Remediation

1. Disable directory indexing for publicly accessible directories.
2. Store sensitive files outside the web root.
3. Restrict access to internal and temporary storage locations.
4. Apply server-side authorization checks before serving files.
5. Review all files currently exposed through the directory.

### 10.2 F-02 — Unauthenticated Access to a Document Marked Confidential

**Affected Resource:** `/ftp/acquisitions.md`
**Finding Category:** Sensitive Information Exposure
**Observed HTTP Status:** `200 OK`
**Verification Status:** Manually Verified

#### Description

A direct unauthenticated HTTP request to `/ftp/acquisitions.md` returned a readable Markdown document.

The document contained the heading `Planned Acquisitions` and explicitly included the statement:

`This document is confidential! Do not distribute!`

The observed response confirms that the document was accessible without authentication in the tested environment.

#### Evidence

- `evidence/raw/12-acquisitions-response.txt`
- `evidence/raw/12-ftp-file-headers.txt`
- `evidence/screenshots/02-sensitive-file-exposure.png`

#### Potential Security Impact

In a real environment, unauthorized access to confidential business documents could expose organizational information.

Potential consequences could include disclosure of business plans, competitive information, or other sensitive material.

The actual sensitivity and authenticity of the document were not independently established.

#### Recommended Remediation

1. Remove confidential documents from publicly accessible directories.
2. Store sensitive files outside the web root.
3. Enforce server-side authentication and authorization.
4. Apply least-privilege access controls to document storage.
5. Review other publicly accessible files for sensitive information.
6. Prevent confidential files from being included in public deployment packages.

### 10.3 Additional Observations — Not Confirmed Vulnerabilities

The following observations were recorded but were not classified as confirmed vulnerabilities.

| Observation | Result | Assessment |
|---|---|---|
| Wildcard CORS header | `Access-Control-Allow-Origin: *` | Requires API-specific testing |
| Missing CSP | Not observed | Hardening opportunity |
| Missing Referrer-Policy | Not observed | Hardening opportunity |
| Missing Permissions-Policy | Not observed | Hardening opportunity |
| Numeric announcement content | HTTP 200 | Encoding or encryption not established |
| `legal.md` | HTTP 200 | Inspected content appeared to be placeholder text |
| Backup-style files | HTTP 403 | Tested resources were denied |
| Nikto path signatures | Mixed | Manual verification identified SPA fallback responses |

## 11. Risk Assessment

Two related exposure conditions were confirmed:

1. Public access to a web directory listing at /ftp/.
2. Unauthenticated retrieval of a document explicitly marked as confidential.

These conditions represent information exposure in the tested application.

A formal CVSS score and final severity rating have not been assigned. A defensible rating would require a documented impact analysis and consideration of the deployment context.

The findings should be prioritized for remediation by restricting public directory access and enforcing authorization on sensitive resources.

## 12. Remediation Summary

The following remediation actions are recommended:

| Priority | Recommended Action |
|---|---|
| High | Remove confidential documents from public web-accessible directories |
| High | Enforce authorization before serving sensitive files |
| High | Disable directory indexing on the exposed directory |
| High | Remove unnecessary files from publicly accessible web directories |
| Medium | Review all publicly accessible files and resources |
| Medium | Configure an appropriate Content Security Policy |
| Medium | Define suitable Referrer-Policy and Permissions-Policy headers |
| Medium | Review CORS configuration on relevant API endpoints |
| Follow-up | Re-test affected resources after remediation |

The priorities above are suggested remediation priorities, not formal CVSS severity ratings.

## 13. Conclusion

The assessment confirmed that OWASP Juice Shop was reachable in the local Docker lab and identified a web-accessible directory listing at the HTTP endpoint /ftp/.

Manual verification also confirmed that a document explicitly marked as confidential could be retrieved without authentication.

Additional tests examined other resources, backup-style files, automated scanner observations, and selected HTTP security headers.

Potential false positives were reviewed and excluded from the confirmed findings when the responses were consistent with the application's single-page application fallback behavior.

The findings and supporting evidence have been documented to provide a traceable record of the assessment.

**Assessment Status:** Testing Completed — Findings Documented.

---

## Appendix A — Evidence Inventory

| Evidence ID | Filename | Description |
|---|---|---|
| E-01 | `01-docker-status.txt` | Docker container status and port mapping |
| E-02 | `02-docker-logs.txt` | Application initialization logs |
| E-03 | `03-nmap.txt` | TCP service discovery |
| E-04 | `04-whatweb.txt` | Web technology identification |
| E-05 | `05-security-headers.txt` | HTTP security header inspection |
| E-06 | `06-http-response.txt` | HTTP response details |
| E-07 | `06-page-title.txt` | Application page title |
| E-08 | `07-nikto.txt` | Automated web reconnaissance |
| E-09 | `08-nikto-manual-verification.txt` | Manual path verification |
| E-10 | `09-robots-body.txt` | Robots exclusion file |
| E-11 | `10-ftp-headers.txt` | Web directory response headers |
| E-12 | `10-ftp-response.html` | Captured web directory listing |
| E-13 | `11-ftp-verification-headers.txt` | Web directory verification headers |
| E-14 | `12-acquisitions-response.txt` | Acquisitions document response |
| E-15 | `12-ftp-file-headers.txt` | Acquisitions response headers |
| E-16 | `13-package-backup-headers.txt` | Package backup response headers |
| E-17 | `13-package-backup-response.txt` | Package backup denial response |
| E-18 | `14-coupons-bak-headers.txt` | Coupons backup response headers |
| E-19 | `14-coupons-bak-denied-response.html` | Coupons backup denial response |
| E-20 | `15-announcement-headers.txt` | Announcement response headers |
| E-21 | `15-announcement-response.txt` | Numeric announcement response |
| E-22 | `16-legal-headers.txt` | Legal document response headers |
| E-23 | `16-legal-response.txt` | Legal document response |
| E-24 | `01-ftp-directory-listing.png` | Web directory listing screenshot |
| E-25 | `02-sensitive-file-exposure.png` | Sensitive document exposure screenshot |

## Appendix B — Evidence Handling

The original raw evidence is retained locally for traceability.

Before publishing this project publicly:

- Review screenshots for confidential text and redact sensitive content where appropriate.
- Do not publish raw confidential document contents unnecessarily.
- Avoid publishing the large numeric announcement response unless required for reproducibility.
- Keep evidence filenames consistent with the references in this report.
- Clearly label illustrative material that is not original assessment evidence.

## Appendix C — Assessment Disclaimer

This assessment was conducted against a deliberately vulnerable application in a local educational lab environment.

The findings describe only the behavior observed during the documented tests. They do not constitute a comprehensive assessment of a production system or any third-party environment.
