---
layout: post
title: "Phising Email Analysis"
author: "Rami Matouk"
date: 2024-11-15
categories: Social Engineering
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
- Does the email content appear malicious?
- Are there any attachments? If so, are they malicious?

This data collection process helps determine the legitimacy of an email and whether it poses a threat.

## Investigation 1
## Investigation 2


By carefully examining these elements, we can better protect ourselves and our organizations from phishing attacks. Stay vigilant, and always verify the authenticity of emails before taking any action. Good luck hunting!
