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

This error message reveals critical information to the attacker, confirming the vulnerability. From here, the attacker can craft a malicious payload, such as: `' OR 1=1 -- -`.
The resulting query would look like this:
``` SQL
SELECT * from users WHERE username = '' OR 1=1 -- -' AND password = 'rami_password';
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
[SQLi payload list](https://github.com/payloadbox/sql-injection-payload-list)

### **Cross-Site Scripting XSS**

Cross-Site Scripting (XSS) is a type of injection-based web vulnerability where malicious scripts are injected into legitimate web applications, allowing them to execute in a user’s browser. 

#### **3 Types of XSS**

1. **Reflected XSS**:  
   A non-persistent form of XSS where the malicious payload is included in the request and executed in the response.

2. **Stored XSS**:  
   A persistent form of XSS where the payload is permanently stored on the web application (e.g., in a database) and executed whenever the relevant data is retrieved.

3. **DOM-Based XSS**:  
   An XSS attack where the payload is executed by modifying the DOM (Document Object Model) in the victim's browser.

#### **How does XSS work?**

Like many other web vulnerabilities, XSS arises from improper data sanitization. Here’s an example:

![image](https://github.com/user-attachments/assets/5c0d70dd-a70c-4bb8-b46b-216e23663845)

The code above displays the value entered in the `user` parameter without sanitization. If a user inputs `LetsDefend` as the parameter, the output looks like this:  

![image](https://github.com/user-attachments/assets/527efd7d-2ccc-4594-916d-dbddbe13f783)

However, if the user inputs malicious code, such as:  
```html
<script>alert(1)</script>
```

![image](https://github.com/user-attachments/assets/4bc6f9a6-8840-442b-ab2c-62ef27850d38)

The browser executes the JavaScript code, and we get a pop-up window. This lack of sanitization is a common way to test for XSS vulnerabilities. Attackers can also use XSS payloads to redirect victims to malicious websites, for example
``` Javascript
<script>window.location=’https://google.com’</script>
```
#### What can an attacker achieve with XSS?
- Steal a user’s session information.
- Capture user credentials.
- Redirect victims to malicious websites.
- Perform other malicious actions.

#### How to Prevent a XSS Vulnerability
1. Monitor for specific keywords: Look for common XSS payloads with keywords such as alert or script.
2. Use frameworks and secure them properly: Utilize frameworks with built-in protection against XSS.
3. Keep frameworks updated: Regularly update frameworks and libraries to address known vulnerabilities.

#### Detecting XSS attack
- Easiest way to detect XSS is by looking for keywords like `alert` or `script`.
- Familiarize yourself with payloads: Learn about common XSS payloads from resources like this [repository](https://github.com/payloadbox/xss-payload-list)
- Look for special characters: Watch for input that includes characters like `<`, `>`, or `()` that could indicate an attempted injection.

With proper sanitization and monitoring, XSS vulnerabilities can be effectively mitigated.

### Command Injection
### Insecure Direct Object Reference (IDOR)
### RFI and LFI
### Open Redirection Attack
### Directory Traversal Attacks
### Brute Force Attacks
### XML External Entity Attacks
