---
title: HackTheBox - Fluffy
date: 2025-07-24
categories: [CTF, HTB]
tags: [Windows, AD, BloodHound, Certipy, active directory,shadow,shadow creds]
---

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/cover.webp)
---
This was a windows box, rated Easy on HTB, pretty straightforward I would say. Beginning with nmap we found smb service, login with creds that were given , we found a pdf with some listed vulnerabilities. We identified one of the CVE, **CVE-2025-24071**, which resided in .library-ms 
file. We captured the hash using responder and with the new creds ran **Bloodhound-py**. Then using **Certipy** we were able to get the user flag, which required compromising WINRM_SVC account. 
Then we again used certipy to write a **Shadow Credential** to get the NTLM hash which lead to exploitation of **ESC16** and thats a way to get root flag.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/fluffy-info.png)

So lets begin with rustscan.
![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/rustscan.webp)
command used : **rustscan -a 10.10.11.69 — -Pn -A**

Since we had port 88 open, we can deduce that this might be an AD box, which means we need Bloodhound to enumerate too.
we updated **/etc/hosts** file with the domain names. 
![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/smb.webp)
 We have smb port open so we ran smbmap to check permissions on different folders. We found we can write in IT folder. Whenever on a Windows box we have write privilage this points to --- **Responder**. We can capture **NTLM hash**!
 Now all we have to do is to find a way.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/pdf.webp)

There was a pdf file in the IT folder which listed some vulnerabilties from which the server suffered from. But there was an intersting **CVE-2025-24071**. As we were reading about it, we found something interesting.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/cve.webp)
And inside the IT folder.
![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/ms-lib.webp)

So we looked into it and there was an IP listed inside the file. I quickly changed my it to mine and used responder to capture the hash.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/responder.webp)

Now we save that hash to a file and crack it using john.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/john.webp)

We got the password of a new account. p.agila and now we can try to run bloodhound-py remotely to get an understanding of AD environment.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/blood.webp)

Then we fired up neo4j and ran bloodhound with the zip file we just captured.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/blood-result.webp)

as we can see in the Abuse column we can add p.agila to Service_Accounts and can write a **Shadow Credential** to get the NTLM hash of this user using **certipy**

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/abuse.webp)

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/shadow.webp)
We got the Hash here and now we can use this to **evil-winrm** and collect te user flag.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/user.webp)

Now we again used certipy to write a **Shadow Credential** to get the NTLM hash of **ca_svc** account

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/shadow2.webp)

now using the creds we just got we can use certipy to find vulnerable templates. For this make sure to use the **latest version** of certipy.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/esc16.webp)
**command**: certipy find -vulnerable -u ca_svc@fluffy.htb -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -stdout

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/update.webp)

Now I’ll request a certificate as ca_operator using the vulnerable template.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/cert.webp)

I’ll cleanup by changing ca_operator’s upn back to what it was.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/update2.webp)

I’ll use the certificate to get the administrator’s NTLM hash.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/admin.webp)

now we get the root flag.

![Alt text for image](/Images/2025-07-23-hackthebox-fluffy/root.webp)
