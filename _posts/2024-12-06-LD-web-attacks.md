---
layout: post
title: "Detecting Common Web Attacks"
author: "Rami Matouk"
date: 2024-12-06
categories: guide
tags: [Blue Team, OWASP, Web]
---

## Introduction
Did you know that 75% of cyberattacks target web applications? In today’s digital landscape, understanding common web vulnerabilities is critical to safeguarding valuable assets and protecting users from harm.

In this post, I’ll break down some of the most prevalent web attacks, explore how attackers exploit these vulnerabilities, and discuss practical ways to prevent or detect them.
- [SQL Injection](#sql-injection)  
- [Cross-Site Scripting (XSS)](#cross-site-scripting-xss)  
- [Command Injection](#command-injection)  
- [Insecure Direct Object Reference (IDOR)](#insecure-direct-object-reference-idor)  
- [Remote File Inclusion (RFI) & Local File Inclusion (LFI)](#rfi-and-lfi)  
- [Open Redirection Attack](#open-redirection-attack)  
- [Directory Traversal Attack](#directory-traversal-attacks)  
- [Brute Force Attacks](#brute-force-attacks)  
- [XML External Entity (XXE) Attack](#xml-external-entity-attacks)  
  
## Open Web Application Security Project
OWASP is non-profit organization dedicated to improving software security. Every few years OWASP publishes list of top 10 vulnerabilites found in production environement. This makes OWASP one of the best sources for improving web application security. 
https://owasp.org/www-project-top-ten/

![image](https://github.com/user-attachments/assets/8a91b43c-4069-44ae-9ee0-f631ae7a5a52)

Currently 2021 list contains these vulnerabilities:
1. Broken Access Control
2. Cryptographic Failures
3. Injection
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable and Outdated Components
7. Identification and Authentication Failures
8. Software and Data Integrity Failures
9. Security Logging and Monitoring Failures
10. Server-Side Request Forgery (SSRF)

The next update of the OWASP Top 10 is expected in the first half of 2025. Until then, the 2021 list remains the go-to framework for prioritizing web security efforts.

## Web Attacks: An In-Depth Exploration
Are you ready to dive into some of the most common web vulnerabilities and how they work? Let’s begin!

### SQL Injection
![image](https://github.com/user-attachments/assets/5d450bcc-f308-4da7-b56c-680a69a5561c)

SQL Injection occurs when an attacker manipulates SQL queries by injecting malicious input. This can result in unauthorized data access, deletion, or even full database compromise. 
- **In-band SQLi**: Query is sent and responded to on the same channel, making it easier for adversaries to exploit.  
- **Inferential SQLi (Blind SQLi)**: The attacker cannot see the direct response but infers information based on application behavior.  
- **Out-of-band SQLi**: Responses are sent through another channel, such as DNS or HTTP requests.

#### **How does it work?**
Login pages are a common entry point for SQL injection attacks. When we log in, the application typically generates an SQL query, for example:  
``` SQL
SELECT * from users WHERE username = 'rami' AND password = 'rami_password';
```
Now, an attacker can manipulate this query by injecting a single quote (`'`), causing an error:
``` SQL
SELECT * from users WHERE username = 'rami'' AND password = 'rami_password';
```
![image](https://github.com/user-attachments/assets/071456c6-5f55-4ba6-88dc-19b9d800157a)

This error message reveals critical information to the attacker, confirming the vulnerability. From here, the attacker can craft a malicious payload, such as: `'OR 1=1; -- -`.
The resulting query would look like this:
``` SQL
SELECT * from users WHERE username = ''OR 1=1; -- -' AND password = 'rami_password';
```
- The `OR` clause ensures the query evaluates to `true`, even if the username is empty (`''`).
- `-- -` comments out the rest of the query.

Because this query always evaluates to true, the attacker gains access to sensitive data, such as the entire users table.

#### What can an attacker achieve?
SQL Injection provides attackers with significant opportunities:
- Authentication bypass
- Command execution
- Exfiltration of sensitive data
- Creating/Deleting/Updating database entries

#### How to prevent SQL Injection?
1. Use secure frameworks: Leverage frameworks that abstract raw SQL queries (e.g., ORM frameworks).
2. Sanitize user input: Never trust user-provided data. Validate and sanitize all inputs, including headers, URLs, and form fields.
3. Avoid raw SQL queries: Use parameterized queries or prepared statements.
4. Keep software updated: Regularly update your frameworks, libraries, and tools.

#### How we detect this attack?
1. Monitor all user input points: This includes form fields, HTTP request headers, and query parameters.
2. Look for SQL keywords: Be vigilant for keywords such as `INSERT`, `SELECT`, `AND`, `OR`, `WHERE`, `UNION`, `JOIN`, etc.
3. Identify special characters: Watch for `'`, `--`, `()` and other characters commonly used in injection attacks.
4. Detect automated SQLi tools:
  - Tools often include their names in the User-Agent header or payload.
  - A high number of rapid requests may indicate automation.
  - Complex payloads are another sign of automated attacks

Familiarizing yourself with common payloads is essential. For an example list, refer to this repository:
https://github.com/payloadbox/sql-injection-payload-list

### Cross Site Scripting XSS
### Command Injection
### Insecure Direct Object Reference (IDOR)
### RFI and LFI
### Open Redirection Attack
### Directory Traversal Attacks
### Brute Force Attacks
### XML External Entity Attacks
