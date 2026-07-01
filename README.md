# SecureBank — Web Application Penetration Testing Project

Security research project demonstrating common web application vulnerabilities in a custom-built banking application. The application was intentionally developed with security flaws and assessed using Burp Suite Community Edition to simulate a real-world penetration test.

---

## Demo

https://github.com/user-attachments/assets/188d22ef-7ed1-4714-9456-320c61b91e3d

---

# Overview

SecureBank is a Node.js and Express banking application created to demonstrate how common web application vulnerabilities can impact financial systems.

After developing the application, I performed a complete penetration test using Burp Suite Community Edition, identifying, exploiting, and documenting multiple security weaknesses. Findings were evaluated against OWASP Top 10 (2021), PCI DSS v4.0, and CVSS v3.1.

This project demonstrates both secure application development concepts and offensive security testing methodologies.

---

# Confirmed Findings

| # | Vulnerability | Severity | Standard | Impact |
|---|---|---|---|---|
| 1 | SQL Injection | 🔴 Critical | OWASP A03 / PCI DSS 6.2.4 | Authentication bypass |
| 2 | Insecure Direct Object Reference (IDOR) | 🟠 High | OWASP A01 / PCI DSS 7.3 | Unauthorized account access |
| 3 | Broken Access Control | 🟠 High | OWASP A01 / PCI DSS 7.3 | Unauthorized fund transfers |
| 4 | Reflected Cross-Site Scripting (XSS) | 🟡 Medium | OWASP A03 / PCI DSS 6.2.4 | Client-side JavaScript execution |

---

# Attack Walkthroughs

## 1. SQL Injection (Critical)

```text
Endpoint: POST /login

Payload:
email=' OR 1=1--

Result:
• Authentication bypassed
• HTTP 302 redirect
• Logged in as Account 1 without valid credentials
```

<img width="600" alt="Login page showing SQL injection payload" src="https://github.com/user-attachments/assets/5b84c51b-7b44-49a3-b234-de20345e2a28" />

*SQL injection payload entered into the login form.*

<img width="600" alt="Burp HTTP History showing POST /login returning 302 redirect to /account/1" src="https://github.com/user-attachments/assets/e96f3bd4-6df5-4cbc-ac0e-5d9dfbc06689" />

*Burp Suite confirms successful authentication bypass.*

---

## 2. Insecure Direct Object Reference (High)

```text
Authenticated User:
Bob (Account ID 2)

Modified Request:
/account/1

Result:
• HTTP 200 response
• Alice's account information returned
```

<img width="600" alt="Burp showing GET /account/1 returning 200 while authenticated as Bob" src="https://github.com/user-attachments/assets/160d805f-3990-4e68-b256-cbac8e33190c" />

*Changing the account ID exposes another user's banking information.*

<img width="600" alt="Burp Repeater showing full account data returned for a different user" src="https://github.com/user-attachments/assets/adbd146a-62f3-4a76-9ad4-17f585c86f77" />

*No server-side authorization check was performed.*

---

## 3. Broken Access Control (High)

```text
Authenticated User:
Bob (Account ID 2)

Modified Parameter:
fromAccountId = 1

Result:
• Transfer processed
• Funds transferred from Alice's account
```

<img width="600" alt="Transfer form with fromAccountId changed to 1" src="https://github.com/user-attachments/assets/fbe24930-d11e-4e1e-9362-0130bf91d64a" />

*Client-controlled account identifier modified before submission.*

<img width="600" alt="Burp showing POST /transfer processed without ownership check" src="https://github.com/user-attachments/assets/56606f06-5414-47eb-96e3-9e359960df53" />

*The server trusted user input instead of validating ownership.*

---

## 4. Reflected Cross-Site Scripting (Medium)

```text
Endpoint:
/search?q=<script>alert(document.cookie)</script>

Result:
• JavaScript executed
• Session cookie exposed
```

<img width="600" alt="XSS payload in search URL" src="https://github.com/user-attachments/assets/053fb83a-4c0d-4b22-95fe-c6293bda3bcd" />

*Malicious JavaScript reflected directly into the page.*

<img width="600" alt="Alert popup confirming JavaScript execution and cookie exposure" src="https://github.com/user-attachments/assets/c866db59-5430-4597-ab4f-f328823200f0" />

*Browser execution confirms successful reflected XSS.*

---

# Security Assessment Summary

The assessment identified four exploitable vulnerabilities affecting authentication, authorization, and input validation.

Key observations include:

- SQL injection allowed authentication without valid credentials.
- Authorization checks were missing across multiple endpoints.
- Server-side trust of client-supplied identifiers enabled unauthorized financial transactions.
- Unsanitized user input allowed arbitrary JavaScript execution within the browser.

Each finding was validated through intercepted HTTP requests and responses captured using Burp Suite Community Edition.

---

# Technology Stack

| Tool | Purpose |
|---|---|
| Node.js + Express | Banking application backend |
| EJS | Server-side templates |
| Burp Suite Community Edition | Web application testing |
| Firefox + FoxyProxy | Proxy configuration and traffic interception |

---

# Testing Methodology

Testing followed established industry guidance, including:

- OWASP Web Security Testing Guide v4.2
- OWASP Top 10 (2021)
- PCI DSS v4.0
- CVSS v3.1 Severity Scoring

A complete penetration testing report is included in:

[SecureBank_Pentest_Report.docx](https://pennstateoffice365-my.sharepoint.com/:w:/g/personal/epj5179_psu_edu/IQD6mwIGJfAISrJfPs1a9I3GAeh3ufORQVinadEkf_RoJBc?e=AYgtfN)

---

# Project Structure

```text
securebank/
├── server.js
├── db/
│   └── fakeDb.js
├── views/
│   ├── login.ejs
│   ├── account.ejs
│   └── search.ejs
├── public/
│   └── style.css
└── .cursor/
    └── mcp.json

SecureBank_Pentest_Report.docx
```

---

# Defensive Recommendations

| Vulnerability | Recommended Mitigation |
|---|---|
| SQL Injection | Use parameterized queries or prepared statements |
| IDOR | Validate ownership on every protected resource |
| Broken Access Control | Derive account information from the authenticated session rather than client input |
| Reflected XSS | Encode output and use escaped EJS tags (`<%= %>`) |

---

## Educational Use

This project was created for cybersecurity education and penetration testing practice. It is intentionally vulnerable and should never be deployed in a production environment.
