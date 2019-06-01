# Connect to an NTFS File Share (Permanently)
Show net neighbors by reviewing ARP cache
```
arp -a
```

Create a new folder and mount it to the NTFS share
```
mkdir /mnt/THRecon
mount -t cifs //192.168.1.120/Linux -o username=myntaccount,password='mypassword' /mnt/THRecon
ls -l /mnt/THRecon
```

Create a credential file to hold the Windows username and password
```
touch /root/.THRecon
chmod 600 /root/.THRecon
ls -l .THRecon
```

```
+++ Begin screen output +++

	-rw------- 1 root root 35 Jul  5 23:59 .THRecon

--- End screen output ---
```

```
vi /root/.THRecon
cat vi /root/.THRecon
```

```
+++ Begin screen output +++

username=my_nt_username
password=my_nt_password

--- End screen output ---
```

Determine the uid and gid of the account that will own the directory after it is mounted
```
whoami
id my_linux_username
```

```
+++ Begin screen output +++

uid=500(my_linux_username) gid=500(my_linux_username) groups=500(my_linux_username)

--- End screen output ---
```

Edit the fstab file (note that uid 500 points to the my_linux_username Linux account)
```
vi /etc/fstab
```

Add a line for THRecon; see output from "cat /etc/fstab" below
```
cat /etc/fstab
```

```
+++ Begin screen output +++

/dev/VolGroup00/LogVol00 /                       ext3    defaults        1 1
LABEL=/boot             /boot                   ext3    defaults        1 2
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
/dev/VolGroup00/LogVol01 swap                   swap    defaults        0 0
//192.168.1.137/THRecon /mnt/THRecon            cifs    auto,credentials=/root/.THRecon,uid=500,gid=500,file_mode=0777,dir_mode=0777 0 0

--- End screen output ---
```


Display the mounted devices
```
mount
```

```
+++ Begin screen output +++

/dev/mapper/VolGroup00-LogVol00 on / type ext3 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
/dev/hda1 on /boot type ext3 (rw)
tmpfs on /dev/shm type tmpfs (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)

--- End screen output ---
```

Reload the fstab file
```
mount -a
```

Display the mounted devices
```
mount
```

```
+++ Begin screen output +++

/dev/mapper/VolGroup00-LogVol00 on / type ext3 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
/dev/hda1 on /boot type ext3 (rw)
tmpfs on /dev/shm type tmpfs (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
THRecon on /mnt/THRecon type cifs (rw,mand)

--- End screen output ---
```

```
ls -l /mnt/THRecon
```

```
+++ Begin screen output +++

total 0
drwxrwxrwx 1 my_linux_username my_linux_username        0 Mar  4 22:10 blah1.csv
-rwxrwxrwx 1 my_linux_username my_linux_username 89712640 Mar  4 22:02 blah2.csv

--- End screen output ---
```