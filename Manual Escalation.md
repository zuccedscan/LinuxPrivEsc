### Search Stored Password From Config files

Try to read config files of available tools of system or check history for finding exposed password and ssh keys.
```sh
find / -type f -name "config" 2>/dev/null # This command will search all config file.
find / -type f -exec grep -il "password" {} 2>/dev/null \; # This command will search every file containing "password" word.
find / -type f -name "config" -exec grep -il "password" {} 2>/dev/null \; # This command do the both things in onmline.
find / -name id_rsa 2> /dev/null # Search ssh keys.
history | grep -i "password"
```

### Escalate Priviledge With ssh Key
If you find any exposed ssh key, try to exploit it for escalation.
```sh
1. Copy and save the key in your own linux terminal with name id_rsa.
2. chmod 400 id_rsa > Change permission.
3. ssh -i id_rsa root@<target-ip> > Try access the root shell of target.
```
