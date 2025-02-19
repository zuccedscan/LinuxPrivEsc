`hostname`: The hostname command will return the hostname of the target machine.
<br>
<br>
`uname -a`: Print system information giving us additional detail about the kernel used by the system.
<br>
<br>
`cat /proc/version`: Looking this file may give you information on the kernel version and additional data such as whether a compiler `(e.g. GCC)` is installed.
<br>
<br>
`cat /etc/issue`: This file usually contains some information about the operating system.
<br>
<br>
`ps -A`: View all running processes.
<br>
`ps axjf`: View process tree.
<br>
`ps aux`: The aux option will show processes for all users.
<br>
<br>
`env`: The env command will show environmental variables.
<br>
<br>
`sudo -l`: The sudo -l command can be used to list all commands your user can run using sudo.
<br>
<br>
`netstat -a`: shows all listening ports and established connections.
<br>
`netstat -at` or `netstat -au`: can also be used to list TCP or UDP protocols respectively.
<br>
`netstat -l`: list ports in “listening” mode.
<br>
`netstat -s`: list network usage statistics by protocol
