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
![image](https://github.com/user-attachments/assets/e8e6fa88-0cba-4f22-9553-b11b8f9f7f1e)

Command injection attacks occur when unsanitized user input is passed directly to the operating system shell, allowing an attacker to execute arbitrary commands.

#### How does Command Injection work?
![image](https://github.com/user-attachments/assets/8577942e-ee16-464a-9438-885465909c5f)

Consider a basic web application that copies a user's file to the `/tmp` folder. Normally, this operation works as intended. However, an attacker could exploit this with a malicious filename such as:  `myfile;ls;.txt` The resulting shell command would be:
``` bash
cp myfile;ls;.txt
```
Here, the ; character separates commands, allowing the attacker to chain multiple commands. For the operating system, this payload executes as:
1. `cp myfile` – Attempts to copy myfile.
2. `ls` – Executes the ls command, listing directory contents.
3. `.txt` – Causes an error since .txt is not a valid command.
Although some commands (myfile and .txt) generate errors, the valid command (ls) executes successfully. While the attacker may not directly see the output, they could use this vulnerability to achieve more damaging results, such as creating a reverse shell to gain full access to the operating system.

#### How to Prevent Command Injection?
- Always sanitize data you receive from a user: Never trust anything you receive from a user. Not even a file name!
- Limit user privileges: Whenever possible, set web application user rights at a lower level. Few web applications require users to have administrator rights. 
- Use virtualization technologies such as dockers.

#### Detecting Command Injection attacks
1. Monitor web requests: Examine all parts of incoming requests, such as query parameters, HTTP headers, and POST data.
2. Look for suspicious keywords: Be on the lookout for commands like dir, ls, cp, cat, type, or other shell-specific commands.
3. Familiarize yourself with payloads: Learn about common command injection payloads. This will help you identify potential exploits more effectively.

Here’s an example of a suspicious request in your access logs:
``` http
GET / HTTP/1.1
Host: yourcompany.com
User-Agent: () { :;}; echo "NS:" $(</etc/passwd)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close
```
In this case, the User-Agent header contains a malicious payload:
``` bash
() { :;}; echo "NS:" $(</etc/passwd)
```
This payload attempts to extract the /etc/passwd file. The contents of the file would be returned to the attacker under the variable NS. Detecting such behavior early is critical to preventing a successful attack.

By implementing proper sanitization, limiting user privileges, and monitoring for malicious patterns, you can reduce the risk of command injection attacks.

### Insecure Direct Object Reference (IDOR)
![image](https://github.com/user-attachments/assets/964733b4-7b6c-431f-ac3f-a9ce7ec3933d)

IDOR, also referred to as "Broken Access Control," is the **#1 web application vulnerability** listed in the 2021 OWASP Top 10.

#### How IDOR Works?
Consider a simple web application that retrieves a parameter like `id` from the user and displays data for the corresponding user:  
``` url
https:/mysite/user?id=1
```
When this request is made, the application displays the data for the user with id=1. But what happens if another user modifies the URL to:
``` url
https://mysite/user?id=2
```
Without proper access controls, the application may display the data of another user. This is a classic example of an IDOR vulnerability, where attackers exploit object references (like id) to access or modify unauthorized data.

Attackers typically exploit IDOR by tampering with parameters such as `id`, `file`, or `order_id` to gain access to restricted resources.

#### How Attackers Take Advantage of IDOR Attacks
- Steal personal information: Extract sensitive user data like emails, passwords, or payment details.
- Access unauthorized documents: Retrieve confidential files, such as invoices or private documents.
- Take unauthorized actions: Perform actions such as deleting or modifying resources belonging to others.

### How to Prevent IDOR?
- Remove unnecessary parameters: Avoid exposing object references like id in URLs or API requests. Instead, use session data to determine the user making the request.
- Enforce proper access controls: Validate whether the user is authorized to access or modify the requested object. Ensure that every access is checked at the server level.

Detecting IDOR Attacks
- Inspect all parameters: Monitor all user-supplied parameters for unusual or malicious attempts to modify values.
- Look for brute-force patterns: Watch for repeated requests to the same endpoint with sequential or predictable values. This may indicate an attacker trying to enumerate resources (e.g., id=1, id=2, id=3).
- Identify suspicious patterns: Notice if requests come in with similar structures, particularly if they involve incremental or predictable parameters.

![image](https://github.com/user-attachments/assets/84cb61c1-1cd8-4507-9c93-0b175c4c4593)

By removing unnecessary references and enforcing strict access control mechanisms, IDOR vulnerabilities can be mitigated effectively.

### RFI and LFI
### Open Redirection Attack
### Directory Traversal Attacks
### Brute Force Attacks
### XML External Entity Attacks
