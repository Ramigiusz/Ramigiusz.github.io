---
layout: post
title: "Phising Email Analysis"
author: "Rami Matouk"
date: 2024-11-30
categories: challenge
tags: [Social Engineering, Phising]
---

![image](https://github.com/user-attachments/assets/d72955a9-f771-4f2c-9103-7826e8e8e203)

Hello, in this post i will analyse 2 phising email attempts to discover how we can find out wether or not email is malicious. We will use Let's Defend Alerts associated with phising attempts for this task.

## A Few Words About Phishing

Phishing is a type of social engineering attack designed to trick users into performing specific actions, such as:
- Downloading malicious attachments.
- Visiting malicious URLs.
- Sharing confidential information.

Attackers employ various techniques to persuade their victims, including:
- **Urgency**: Attackers create a sense of urgency to pressure users into acting quickly. For example, they might claim that the recipient has only 24 hours to click a link or risk losing access to their account.
- **Authority**: Phishing emails often impersonate trusted entities, such as banks or well-known companies, by replicating their branding and tone.
- **Deceptive URLs**: Attackers craft fake websites with addresses that closely resemble legitimate ones. These subtle changes are easy to overlook, especially for inexperienced users—for example, `www.paypai.com` instead of `www.paypal.com.`

### Types of Phishing Emails
Phishing comes in various forms, including:
- **Spear Phishing**: A targeted attack against a specific individual. This type of phishing begins with extensive reconnaissance to gather information about the victim. Spear phishing is highly effective, with success rates reported as high as 91%.
- **Whaling**: A subtype of spear phishing aimed at upper management or high-ranking executives in an organization. The stakes are higher, as the attackers target individuals with greater access to sensitive information or decision-making power.
- **Clone Phishing**: Attackers use a legitimate email as a template to create a malicious copy. They replace genuine URLs and attachments with malicious ones and may use compromised accounts to send the email, making it appear more credible.

## Art of Investigation
When investigating a phishing email, it is essential to collect and analyze critical information, such as:
- When was the email sent?
- What is the SMTP address?
- Who is the sender?
- Who is the recipient?
- Device Action - allowed or blocked?
- Does the email content appear malicious?
- Are there any attachments? If so, are they malicious?

This data collection process helps determine the legitimacy of an email and whether it poses a threat.

## Investigation 1
![image](https://github.com/user-attachments/assets/1a6bf323-2c83-4449-bee2-597a34accb1b)

We received an alert labeled "SOC120 - Phishing Mail Detected - Internal to Internal". Our task is to investigate this potential threat. As previously mentioned, we need to collect crucial data about the email. Based on the alert overview, here’s what we know so far:
1. When was the email sent? - Feb, 07, 2021, 04:24 AM
2. What is the SMTP address? - 172.16.20.3 <- Internal IP address
3. Who is the sender? - john@letsdefend.io
4. Who is the recipient? - susie@letsdefend.io
5. Action - Allowed

To further analyze the email content and any potential attachments, we need to examine the email itself. I navigated to the Email Security tab in the Let's Defend SOAR environment and searched for emails sent by john@letsdefend.io.

### Email Details

![image](https://github.com/user-attachments/assets/e1f60e70-1853-48f4-a42a-db07480111cf)


![image](https://github.com/user-attachments/assets/1ff34442-2063-47c3-b1ff-b7262c81edc3)

6. Does the email content appear malicious?
Upon inspection, the email content shows that John is inviting Susie on a date. There are no URLs, phishing attempts, or other suspicious indicators in the message
7. Are there any attachments? If so, are they malicious?
No, the email contains no attachments.

Based on the information gathered, this email does not appear to be malicious. To confirm, I proceeded to the Case Management section and followed the playbook to validate my findings.

![image](https://github.com/user-attachments/assets/4cdebf4a-db38-4878-ae18-9cb63e8578ce)

![image](https://github.com/user-attachments/assets/4e9d4a0e-29fa-4bec-a0e2-bc062030dfe2)

![image](https://github.com/user-attachments/assets/ce354298-1c80-48eb-9c99-5614a8fead6c)

![image](https://github.com/user-attachments/assets/464f3f22-204b-401b-91e1-451f2fd411e1)

![image](https://github.com/user-attachments/assets/b10ee387-29e5-44f1-bdc8-8c8a39af53de)

![image](https://github.com/user-attachments/assets/badf7591-2e4e-428e-bca5-2318d7eb5116)

### Conclusion
It turns out my initial assumptions were correct. This was an innocent email with no malicious intent (or so we hope! 😂). This particular case was a straightforward investigation, but it’s important to remain vigilant and follow standard procedures for every alert to ensure nothing suspicious slips through.

## Investigation 2
We received another alert, this time labeled "SOC114 - Malicious Attachment Detected - Phishing Alert". Based on the initial analysis, it appears to be a dangerous phishing attempt. Let’s investigate further.
1. When was the email sent? - Jan, 31, 2021, 03:48 PM
2. What is the SMTP address? - 49.234.43.39 <- External IP address
3. Who is the sender? - accounting@cmail.carleton.ca
4. Who is the recipient? - richard@letsdefend.io
5. Action - The email was allowed to pass through.

### Email Analysis
I navigated to the Email Security tab in the Let's Defend SOAR environment and searched for emails sent by accounting@cmail.carleton.ca.

6. Does the email content appear malicious?
The email is suspicious due to its overly simplistic message, which often characterizes phishing attempts.
A VirusTotal scan of the sender's IP address flagged it as suspicious:

![image](https://github.com/user-attachments/assets/80da3a9f-df15-4b22-9a1d-a5fb12917f10)

7. Are there any attachments? If so, are they malicious?
The email contains an attachment. The attachment was analyzed using Hybrid Analysis and identified as malicious. It exploits CVE-2017-11882, a vulnerability in Microsoft Office Service Packs.

![image](https://github.com/user-attachments/assets/ffb96cc9-9239-4f8d-a476-e21fea7d1cb8)

VirusTotal also highlighted related indicators, such as IP addresses and malicious websites that the malware connects to:

![image](https://github.com/user-attachments/assets/4a19df3a-966b-4467-8b18-914de9325731)

### Action Taken:
This investigation confirms that the email is malicious. Since it was allowed to pass through the system, immediate remediation steps are required:
1. Email removal: The email must be deleted from the recipient’s mailbox to prevent further risk.
2. User Activity Analysis
Under the Endpoints tab, I discovered that the user's device accessed one of the malicious websites, indicating they likely opened the attachment:

![image](https://github.com/user-attachments/assets/6ce534a8-0cd2-452a-8da4-a9758914cd1c)

The user’s endpoint is confirmed to be infected and must be contained immediately to prevent lateral movement within the network.

![image](https://github.com/user-attachments/assets/b06a86f2-0b7a-4e68-b90c-3e988415c8dc)

With all the data collected, I documented the findings and proceeded to close the alert after completing all the required actions:

![image](https://github.com/user-attachments/assets/8f880913-71e0-4ae8-b623-a38131f36026)

### Conclusion
This investigation highlights the critical nature of responding to phishing alerts promptly. Despite the email bypassing initial filters, proper monitoring, analysis, and response allowed us to mitigate the threat before further damage occurred.
