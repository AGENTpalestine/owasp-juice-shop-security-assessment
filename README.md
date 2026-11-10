# OWASP Juice Shop — Web Application Security Assessment

Web Security Assessment | Manual Verification | Security Evidence

## Project Overview

This project documents a hands-on web application security assessment of OWASP Juice Shop, a deliberately vulnerable web application deployed in a local Docker-based lab environment.

The assessment covered service discovery, technology identification, HTTP response analysis, security header review, automated reconnaissance, and manual verification of potentially exposed resources.

Two exposure conditions were confirmed:

1. A publicly accessible web directory listing that disclosed resource names.
2. Unauthenticated access to `acquisitions.md`, a document explicitly marked as confidential.

The document's authenticity and actual business sensitivity were not independently established. Automated scanner observations were manually reviewed to distinguish confirmed findings from false positives.

**Assessment Status:** Testing Completed — Findings Documented.

## Assessment Environment

| Component | Details |
|---|---|
| Target Application | OWASP Juice Shop |
| Application Version | 20.3.0 |
| Operating System | Kali Linux |
| Container Engine | Docker 28.5.2 |
| Runtime | Node.js v24.21.0 |
| Container | `juice-shop` |
| Application Port | TCP 3000 |
| Target URL | `http://127.0.0.1:3000` |
| Deployment | Local Docker Container |
| Environment | Local Educational Lab |
| Assessment Period | November 9–10, 2026 |

## Assessment Objectives

- Verify application deployment and availability.
- Identify exposed network services.
- Inspect web application technologies and HTTP responses.
- Identify publicly accessible directories and files.
- Manually verify potentially sensitive resources.
- Review selected HTTP security headers.
- Distinguish confirmed findings from scanner false positives.
- Document evidence and recommend remediation.

## Key Findings

| ID | Finding | Description |
|---|---|---|
| F-01 | Unauthenticated Web Directory Listing | The `/ftp/` web path exposed a directory listing without authentication. |
| F-02 | Unauthenticated Access to a Document Marked Confidential | The `acquisitions.md` document was accessible without authentication and contained a confidentiality notice. |

## Methodology

1. Application and container availability verification.
2. Network service discovery and enumeration.
3. Web technology identification.
4. HTTP response and security header inspection.
5. Automated reconnaissance.
6. Manual verification of exposed resources.
7. Evidence collection and technical documentation.
8. Security remediation recommendations.

## Tools & Technologies

- Kali Linux
- Docker
- Nmap
- WhatWeb
- Nikto
- cURL
- Linux Command Line
- Markdown

## Repository Structure

```text
.
├── README.md
├── .gitignore
├── .markdownlint.json
├── report/
│   └── Technical-Assessment-Report.md
└── evidence/
    └── sanitized/  # Selected reviewed evidence
```

Only selected evidence reviewed for public release is included. Original raw evidence is retained separately and excluded from the public repository.

## Technical Report

The full technical assessment report contains the scope, methodology, observations, findings, supporting evidence inventory, risk considerations, and remediation recommendations.

[Read the Technical Assessment Report](report/Technical-Assessment-Report.md)

## Scope Disclaimer

OWASP Juice Shop is an intentionally vulnerable application used for educational security testing.

All assessment activities were performed against a locally deployed lab instance.

This project documents the tested endpoints and observed behavior only. It does not represent a comprehensive assessment of every application feature or API endpoint, nor does it establish the security posture of any production deployment.

---

Web Application Security Assessment | Cybersecurity Portfolio
