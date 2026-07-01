# SecureBank — Web Application Penetration Test
 
Security research project demonstrating real-world web application vulnerabilities in a custom-built intentionally vulnerable fintech banking app — tested using Burp Suite Community Edition.
 
---
 
## Demo
 
https://github.com/user-attachments/assets/188d22ef-7ed1-4714-9456-320c61b91e3d
 
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
 
<img width="600" alt="Login page showing SQL injection payload" src="https://github.com/user-attachments/assets/5b84c51b-7b44-49a3-b234-de20345e2a28" />
<img width="600" alt="Burp HTTP History showing POST /login returning 302 redirect to /account/1" src="https://github.com/user-attachments/assets/e96f3bd4-6df5-4cbc-ac0e-5d9dfbc06689" />
---
 
### 2 — IDOR (High)
 
```
Logged in as: Bob (Account ID 2)
Visited:      /account/1
Result:       HTTP 200 — Alice's full balance and
              transaction history returned
```
 
<img width="600" alt="Burp showing GET /account/1 returning 200 while authenticated as Bob" src="https://github.com/user-attachments/assets/160d805f-3990-4e68-b256-cbac8e33190c" />
<img width="600" alt="Burp Repeater showing full account data returned for a different user" src="https://github.com/user-attachments/assets/adbd146a-62f3-4a76-9ad4-17f585c86f77" />
---
 
### 3 — Broken Access Control (High)
 
```
Logged in as: Bob (Account ID 2)
Modified:     fromAccountId = 1 in transfer form
Result:       Server processed transfer from Alice's
              account without authorisation check
```
 
<img width="600" alt="Transfer form with fromAccountId changed to 1" src="https://github.com/user-attachments/assets/fbe24930-d11e-4e1e-9362-0130bf91d64a" />
<img width="600" alt="Burp showing POST /transfer processed without ownership check" src="https://github.com/user-attachments/assets/56606f06-5414-47eb-96e3-9e359960df53" />
---
 
### 4 — Reflected XSS (Medium)
 
```
Endpoint: GET /search?q=<script>alert(document.cookie)</script>
Result:   JavaScript executed in browser
          Session cookie exposed via alert popup
```
 
<img width="600" alt="XSS payload in search URL" src="https://github.com/user-attachments/assets/053fb83a-4c0d-4b22-95fe-c6293bda3bcd" />
<img width="600" alt="Alert popup confirming JavaScript execution and cookie exposure" src="https://github.com/user-attachments/assets/c866db59-5430-4597-ab4f-f328823200f0" />
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
Full findings documented in the [pentest report]((https://pennstateoffice365-my.sharepoint.com/:w:/g/personal/epj5179_psu_edu/IQD6mwIGJfAISrJfPs1a9I3GAeh3ufORQVinadEkf_RoJBc?e=zHGae4).
 
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
