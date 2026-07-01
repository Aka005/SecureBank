# SecureBank — Web Application Penetration Test
 
Security research project demonstrating real-world web application vulnerabilities in a custom-built intentionally vulnerable fintech banking app — tested using Burp Suite Community Edition.
 
---
 
## Demo
 
> https://github.com/user-attachments/assets/188d22ef-7ed1-4714-9456-320c61b91e3d


---
 
## What I Built
 
Designed and developed **SecureBank** — a deliberately vulnerable Node.js banking application simulating a consumer-facing fintech portal. Then conducted a full penetration test against it, documenting every finding in a professional report following OWASP and PCI DSS standards.
 
This project demonstrates both the **builder** and **breaker** mindset — I planted the vulnerabilities, then found and exploited them the way a real attacker would.
 
---
 
## Findings
 
| # | Vulnerability | Severity | Standard | Result |
|---|---|---|---|---|
| 1 | SQL Injection on login endpoint | 🔴 Critical | OWASP A03 / PCI DSS 6.2.4 | Auth bypassed — no credentials needed |
| 2 | IDOR on account page | 🟠 High | OWASP A01 / PCI DSS 7.3 | Any user can view any account |
| 3 | Broken access control on transfers | 🟠 High | OWASP A01 / PCI DSS 7.3 | Funds transferred from another user's account |
| 4 | Reflected XSS in search | 🟡 Medium | OWASP A03 / PCI DSS 6.2.4 | Session cookie stolen via crafted URL |
 
---
 
## Attack Walkthroughs
 
### 1 — SQL Injection (Critical)
```
Endpoint: POST /login
Payload:  email = ' OR 1=1--
Result:   HTTP 302 → /account/1
          Authenticated as Alice with no valid password
```
![SQLi login page](images/sqli_login.png)
![SQLi Burp history](<img width="848" height="407" alt="image" src="https://github.com/user-attachments/assets/092e9176-7aea-4ef8-8f82-06e697b75e2e" />)


 
---
 
### 2 — IDOR (High)
```
Logged in as: Bob (Account ID 2)
Visited:      /account/1
Result:       HTTP 200 — Alice's full balance and
              transaction history returned
```
![IDOR Burp](images/idor_burp.png)
![IDOR URL bar](images/idor_url.png)
 
---
 
### 3 — Broken Access Control (High)
```
Logged in as: Bob (Account ID 2)
Modified:     fromAccountId = 1 in transfer form
Result:       Server processed transfer from Alice's
              account without authorisation check
```
![Transfer form tampered](images/transfer_form.png)
![Transfer Burp](images/transfer_burp.png)
 
---
 
### 4 — Reflected XSS (Medium)
```
Endpoint: GET /search?q=<script>alert(document.cookie)</script>
Result:   JavaScript executed in browser
          Session cookie exposed via alert popup
```
![XSS URL](images/xss_url.png)
![XSS alert popup](images/xss_alert.png)
 
---
 
## Key Findings
 
**SQL injection succeeded with no prior access** — a single unauthenticated request bypassed all authentication and returned a valid session for the first account in the database.
 
**Broken access control is systemic** — both the account page and transfer endpoint trust client-supplied IDs instead of the server session. This is a design flaw, not just a single bug.
 
**All four vulnerabilities confirmed via Burp Suite** — every finding includes HTTP request/response evidence captured through the Burp proxy.
 
---
 
## Stack
 
| Tool | Purpose |
|---|---|
| Node.js + Express | Banking app backend |
| EJS | Templating (intentionally unescaped for XSS) |
| Burp Suite Community | Traffic interception and exploitation |
| Firefox + FoxyProxy | Browser proxy routing |
 
---
 
## Methodology
 
Testing followed industry standards for financial application security:
 
- **OWASP Testing Guide v4.2**
- **OWASP Top 10 (2021)**
- **PCI DSS v4.0 — Requirement 11.4**
- **CVSS v3.1** for severity scoring
Full findings documented in the [pentest report](SecureBank_Pentest_Report.docx).
 
---
 
## Files
 
```
securebank/
├── server.js              # Express app with intentional vulnerabilities
├── db/fakeDb.js           # In-memory DB layer (SQLi simulation)
├── views/                 # EJS templates
│   ├── login.ejs          # Vulnerable login form
│   ├── account.ejs        # Vulnerable account page + transfer form
│   └── search.ejs         # Vulnerable search (unescaped XSS)
├── public/style.css       # Styling
└── .cursor/mcp.json       # MCP config (used in separate MCP research)
 
SecureBank_Pentest_Report.docx   # Full professional pentest report
```
 
---
 
## Defenses
 
| Vulnerability | Fix |
|---|---|
| SQL Injection | Parameterised queries — never concatenate user input into queries |
| IDOR | Server-side ownership checks on every account endpoint |
| Broken access control | Derive source account from session, never from request body |
| Reflected XSS | Encode all output — use `<%= %>` not `<%- %>` in EJS |
 
---
 
*Built for educational and security research purposes only. Do not deploy publicly.*
