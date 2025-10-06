# Summary
- link https://tryhackme.com/room/w1seguy
- best reference reading: https://www.101computing.net/xor-encryption-algorithm/
- learning: XOR is reversible

# Write-up to get the flag
- it was indicated as an easy room that couldn't be finished unless there was a crystal understanding on how XOR works.
- for those who didn't understand XOR I would suggest the the link attached in the summary.
- by checking the source code given in the room, and the challgen given, one could tell it was a task to decrypt and find the encryption key. Here was the code given and assumption something similiar was running in machine.
```python
import random
import socketserver 
import socket, os
import string

flag = open('flag.txt','r').read().strip()

def send_message(server, message):
    enc = message.encode()
    server.send(enc)

def setup(server, key):
    flag = 'THM{thisisafakeflag}' 
    xored = ""

    for i in range(0,len(flag)):
        xored += chr(ord(flag[i]) ^ ord(key[i%len(key)]))

    hex_encoded = xored.encode().hex()
    return hex_encoded

def start(server):
    res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))
    key = str(res)
    hex_encoded = setup(server, key)
    send_message(server, "This XOR encoded text has flag 1: " + hex_encoded + "\n")
    
    send_message(server,"What is the encryption key? ")
    key_answer = server.recv(4096).decode().strip()

    try:
        if key_answer == key:
            send_message(server, "Congrats! That is the correct key! Here is flag 2: " + flag + "\n")
            server.close()
        else:
            send_message(server, 'Close but no cigar' + "\n")
            server.close()
    except:
        send_message(server, "Something went wrong. Please try again. :)\n")
        server.close()

class RequestHandler(socketserver.BaseRequestHandler):
    def handle(self):
        start(self.request)

if __name__ == '__main__':
    socketserver.ThreadingTCPServer.allow_reuse_address = True
    server = socketserver.ThreadingTCPServer(('0.0.0.0', 1337), RequestHandler)
    server.serve_forever()
```
Here was the question from the machine.
<img width="697" height="123" alt="Screenshot from 2025-10-06 16-17-14" src="https://github.com/user-attachments/assets/c4694b1b-2a81-415a-8759-62be361cebf2" />

- the situation was, we had `Flag XOR Key = Cipher`. We were given the Cipher, how to find the KEY?
- Few more things we knew or had to assume, as per the information given were:
  - the KEY in ascii has 5 characters as per the code `res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))`
  - the KEY was a combination of letters and digits, low and cap.
  - Looking at the Answer format, we could assume the flag was in the usual format `THM{........}`.
- <img width="653" height="449" alt="Screenshot from 2025-10-06 16-11-52" src="https://github.com/user-attachments/assets/4012fdee-46f7-48ad-865f-fd6f47705bc0" />
- The Cipher is hex encoded as per the python `hex_encoded = xored.encode().hex()`.
    - xored.encode().hex() converts each string/bytes to 2 hexadecimal characters
  - Since XOR is reversible (refer to reference if required),we can use XOR in cyberchef to find the KEY first 4 characters of the KEY, and use manual brute force the last character/digit. These are reversible:
    - FLAG xor KEY = Cipher
    - FLAG xor Cipher = KEY
    - KEY xor Cipher = FLAG
- the XOR encoded in my case was 2722202d (4 bytes).
- the FLAG started with THM{ (4 bytes).
- Put these in cyberchef (FLAG xor Cipher = KEY). We had the first 4 character of the KEY : cjmV
- <img width="1862" height="571" alt="Screenshot from 2025-10-06 16-20-54" src="https://github.com/user-attachments/assets/9491fe05-ec44-436f-b609-0489dec5eeed" />
- Next, put the full Cipher as Input. Remember this was HEX encoded so need to use CyberCheft to decode (from HEX). Then applied XOR (used the first 4-char obtained as the KEY - in UTF8. One would be able to see something similiar.
- The finaly part was to manually brute force the last character of the KEY withg 1-0 a-z A-Z until the output had the last character showing `}`.
- In my case it was cjmV8. Yours would be different. The output was the first answer. In this case we did : KEY xor Cipher = FLAG
- <img width="1708" height="492" alt="image" src="https://github.com/user-attachments/assets/f035b73f-f851-41bc-b788-c3760b39bc34" />

- The last part was to key in the KEY obtained to to get the 2nd answer.

