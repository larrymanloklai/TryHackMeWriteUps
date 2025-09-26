# 1) Summary:
- Room:   Include
- URL:    https://tryhackme.com/room/include
- Skills: SSRF, file inclusion, path transversal, prototype pollution, server-side attacks.
- Help:   If anyone can poison the SSH log and get RCE please share thanks. I could only inject base64 encoded but without php wrapper working I couldn't get the RCE.

# 2) Writeups to get the flags:
- After starting the target machine. nmap was started to do port discovery
  <img width="650" height="423" alt="Screenshot from 2025-09-26 16-50-04" src="https://github.com/user-attachments/assets/5969efe3-de2d-4ffa-a6fa-0b939d8693b0" />

- As per the service we can see the standard SSH and mail server. We can also see special port 4000 and 50000. It was folllowed up by ` nmap -sS -sV -O -sC -p22,25,110,143,993,995,4000,50000 10.201.23.40 ` to further enumerate, and visited port 4000 and 50000 in web browser, proxy via Burp as it might be required. The login name and password is given in the page as `username: guest, password: guest`
- In the Friend Details page, there is an area for input. I tried test and test, and found a property "test" was added with value "test". I further tried and I could chanage the age. So I tried to change isAdmin to true:
  <img width="849" height="828" alt="Screenshot from 2025-09-26 16-57-57" src="https://github.com/user-attachments/assets/fa027f35-4246-4adf-bad5-eecc8f5b7bb1" />
- it worked. And at the same time additional manual in the upper right hand corner appears.
    <img width="748" height="796" alt="Screenshot from 2025-09-26 17-01-32" src="https://github.com/user-attachments/assets/0089f835-cee1-4953-9986-8c507de42b3c" />
- The API page revealed a list of important APIs accessible to admins. Noted and a point for SSRF attack is required
- After further enumeration, it was found the Setting Page appears to be an easy point for SSRF. I put `http://127.0.0.1:5000/internal-api` as the banner image URL, and I received the outcome.
```
 data:application/json; charset=utf-8;base64,eyJzZWNyZXRLZXkiOiJzdXBlclNlY3JldEtleTEyMyIsImNvbmZpZGVudGlhbEluZm8iOiJUaGlzIGlzIHZlcnkgY29uZmlkZW50aWFsIGluZm9ybWF0aW9uLiBIYW5kbGUgd2l0aCBjYXJlLiJ9
```
- Then I also tried the admin API `http://127.0.0.1:5000/getAllAdmins101099991`. it also worked.
```
data:application/json; charset=utf-8;base64,eyJSZXZpZXdBcHBVc2VybmFtZSI6ImFkbWluIiwiUmV2aWV3QXBwUGFzc3dvcmQiOiJhZG1pbkAhISEiLCJTeXNNb25BcHBVc2VybmFtZSI6ImFkbWluaXN0cmF0b3IiLCJTeXNNb25BcHBQYXNzd29yZCI6IlMkOSRxazZkIyoqTFFVIn0=
```
- These are base64 code. I encoded these in cyberchef https://gchq.github.io/CyberChef/, and obtained the username and password
<img width="1027" height="807" alt="image" src="https://github.com/user-attachments/assets/0e26ef2b-12c1-4702-8e72-3ad744fa5e92" />
- Using the username and password to login to port 50000 to abtain the first flag.
<img width="1003" height="389" alt="image" src="https://github.com/user-attachments/assets/9449131c-6618-42af-b5a6-2117a96aca42" />
- there appears to be nothing else on the surface. Upon checking the source code a point of file inclusion / path transveral is possible a point of entry - `control U` : see line 32 `profile.php?img=` could be an entry point.
<img width="993" height="666" alt="Screenshot from 2025-09-26 17-14-39" src="https://github.com/user-attachments/assets/04a489ff-2f5b-4a91-935a-ce817a768721" />
- These were tried:
`../../../../../../../../../../../../../../../../../../../../proc/version`
`..//..//..//..//..//..//..//..//..//..//..//..//..//..//..//..//..//..//..//..//proc/version`
`....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//proc/version`
- with `....//` it bypassed the measurement - proof of concept was done. Payload was changed to /etc/passed and it also worked. Noted normally or generally numbers of `....//` doesn't matter.
<img width="993" height="402" alt="Screenshot from 2025-09-26 17-20-46" src="https://github.com/user-attachments/assets/5142d589-c953-4006-8a73-7e38c23d82e0" />
- users charles and joshua were found. I proceed to try cracking their password using hydra. The second flag was found.
- <img width="1125" height="858" alt="image" src="https://github.com/user-attachments/assets/8343ce57-0950-4d2a-b86d-791faa31ea20" />

# 3) Writeups on additional learnings:
- I was not satisfied how the second flag was obtained by hydra brute force. So I continued to exploit vulunability to get RCE from LFI.
- php and data wrapper were tried and I couldn't get any outcome e.g. `http://10.201.118.205:50000/profile.php?img=data://text/plain,HelloWorld`, `data://text/plain,<?php phpinfo(); ?>` etc. 
- So I proceed to google and find out where are the commone apache2 error log, ssh log and mail log located. As per the enumeration from nmap these are my target. But prior to that I also generated some login error via SSH, mail and server. These were what I tried.
  
1) var/log/apache2/access.log,
2) var/log/apache2/error.log,
3) var/log/httpd-access.log,
4) var/log/httpd/access_log,
5) var/log/auth.log,
6) var/log/secure,
7) var/log/mail.log,
8) var/log/maillog,

- the result was: auth.log and mail.log are subject to log poisioning.
- First I tried to poision the auth.log. I could only inject base64 encoded (successful); however, without a php wrapper running I wasn't able to trigger RCE.
<img width="878" height="419" alt="Screenshot from 2025-09-26 18-25-15" src="https://github.com/user-attachments/assets/fc0367c7-5ab0-4243-963c-86decbc52360" />
- Second I tried the mail.log. Here was how the payload was injected. A 501 was enough to log the payload.
<img width="737" height="184" alt="image" src="https://github.com/user-attachments/assets/2e0e77ce-bef4-4296-8877-6d2269dfcfae" />
- Then I went back to trigger the payload `var/log/mail.log&cmd=ls -la /var/www/html`. Found the higlighted
- <img width="1156" height="649" alt="Screenshot from 2025-09-26 18-50-17" src="https://github.com/user-attachments/assets/8da76ed3-c903-4f1a-ba52-e46769a1f47d" />
- finally foudn the flag: `var/log/mail.log&cmd=cat /var/www/html/505eb0fb8a9f32853b4d955e1f9123ea.txt`
<img width="1156" height="576" alt="image" src="https://github.com/user-attachments/assets/db1495a8-cbf5-4f25-a10d-6805d4e74221" />







  
  

