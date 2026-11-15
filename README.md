# Web Application VAPT — OWASP Juice Shop

A hands-on Vulnerability Assessment & Penetration Test (VAPT) of [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/), performed as a portfolio project to demonstrate practical web application security testing skills for Cyber Security / SOC / VAPT internship applications.

📄 **[Full VAPT Report (PDF)](./VAPT_Report_OWASP_JuiceShop.pdf)**

---

## 🎯 Objective

Simulate a real-world black-box/grey-box web application penetration test against an intentionally vulnerable application, using the same methodology and tooling used by professional pentesters/SOC analysts:

- Reconnaissance → Mapping → Exploitation → Reporting
- Manual exploitation (no automated scanners relied upon for findings)
- Professional report with evidence, root cause, and remediation for every finding

## 🧰 Tools Used

| Tool | Purpose |
|---|---|
| [OWASP Juice Shop](https://github.com/juice-shop/juice-shop) | Target application (run locally on `localhost:3000`) |
| [Burp Suite Community Edition](https://portswigger.net/burp/communitydownload) | Intercepting proxy, manual request tampering (Repeater) |
| [Nmap](https://nmap.org/) | Port & service reconnaissance |
| `curl` | HTTP header / CORS analysis |

## 🖥️ Environment

- Target: OWASP Juice Shop v20.4.0-SNAPSHOT, running locally via `npm start`
- Node.js v24.22.3 LTS
- Windows 11
- Burp Suite Community Edition v2026.10.1
- Nmap 7.993

> ⚠️ All testing was performed exclusively against a **local, self-hosted instance** on `127.0.0.1`. No external, production, or third-party systems were targeted at any point.

## 🔍 Methodology

1. **Reconnaissance** — Nmap port/service scan, curl header analysis
2. **Mapping** — Passive endpoint discovery via Burp Suite HTTP History while browsing the app
3. **Exploitation** — Manual testing and exploitation of discovered weaknesses
4. **Reporting** — Evidence capture, OWASP Top 10 mapping, severity rating, remediation

## 📋 Findings Summary

| # | Finding | OWASP Category | Severity |
|---|---|---|---|
| 1 | Hidden admin page disclosed via client-side JS route table | A05:2021 – Security Misconfiguration | 🟦 Low |
| 2 | SQL Injection — full authentication bypass (`'OR 1=1--`) | A03:2021 – Injection | 🟥 **Critical** |
| 3 | DOM-based XSS via search parameter (`<iframe src="javascript:alert(...)">`) | A03:2021 – Injection | 🟧 High |
| 4 | IDOR — read another user's basket by incrementing `/rest/basket/{id}` | A01:2021 – Broken Access Control | 🟧 High |
| 5 | Sensitive data exposure via public `/ftp` directory (confidential docs, `.bak` files, `.kdbx` vault) | A01 / A05:2021 | 🟨 Medium |
| 6 | Permissive CORS (`Access-Control-Allow-Origin: *`) & missing CSP header | A05:2021 – Security Misconfiguration | 🟨 Medium |

Full details — steps to reproduce, evidence screenshots, root cause analysis, and remediation guidance for each finding — are in the **[PDF report](./VAPT_Report_OWASP_JuiceShop.pdf)**.

## 🏆 Highlight: SQL Injection → Admin Authentication Bypass

```
POST /rest/user/login
{"email":"'OR 1=1--","password":"x"}
```

This single payload bypassed the login's SQL query logic entirely, returning a valid JWT for `admin@juice-sh.op` with no knowledge of any password — a complete compromise of the authentication layer.

## 📝 Key Takeaways

- Obscuring functionality in the UI (hiding a route) is not a substitute for server-side authorization.
- String-concatenated SQL queries remain one of the most severe and easiest-to-exploit vulnerability classes.
- Authentication ≠ Authorization — IDOR showed that a valid token alone was enough to access another user's private data, because ownership was never checked.
- Defense-in-depth headers (CSP, strict CORS) matter even when other bugs exist, since they reduce the real-world blast radius of an XSS or injection flaw.

## 📚 How to Reproduce

```bash
# 1. Clone and run Juice Shop locally
git clone https://github.com/juice-shop/juice-shop.git
cd juice-shop
npm install
npm start
# App runs on http://localhost:3000

# 2. Route traffic through Burp Suite (listening on 127.0.0.1:8080)
# 3. Follow the steps documented in the PDF report for each finding
```

## ⚖️ Disclaimer

This project was conducted solely against a local instance of OWASP Juice Shop, an application explicitly published by OWASP for security training and practice. No real-world or production systems were tested. This repository is for educational/portfolio purposes only.

---

**Author:** Anushka Sengupta — B.Sc. Cyber Security, Guru Nanak Institute of Technology, Kolkata
