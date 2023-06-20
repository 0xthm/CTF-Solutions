# Enumeration

Scanned the machine using this #nmap command

```bash

nmap -sC -sV -T4 "IP MACHINE"

```

we have got this result 

```bash

Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-02 12:23 EDT

Nmap scan report for 10.10.90.135

Host is up (0.14s latency).

Not shown: 996 closed tcp ports (reset)

PORT    STATE SERVICE VERSION

21/tcp  open  ftp     vsftpd 3.0.2

22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)

| ssh-hostkey: 

|   1024 5650bd11efd4ac5632c3ee733ede87f4 (DSA)

|   2048 396f3a9cb62dad0cd86dbe77130725d6 (RSA)

|   256 a66996d76d6127967ebb9f83601b5212 (ECDSA)

|_  256 3f437675a85aa6cd33b066420491fea0 (ED25519)

80/tcp  open  http    Apache httpd

|_http-title: Purgatory

|_http-server-header: Apache

111/tcp open  rpcbind 2-4 (RPC #100000)

| rpcinfo: 

|   program version    port/proto  service

|   100000  2,3,4        111/tcp   rpcbind

|   100000  2,3,4        111/udp   rpcbind

|   100000  3,4          111/tcp6  rpcbind

|   100000  3,4          111/udp6  rpcbind

|   100024  1          40754/tcp6  status

|   100024  1          45927/udp   status

|   100024  1          49938/udp6  status

|_  100024  1          51976/tcp   status

Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel



Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

Nmap done: 1 IP address (1 host up) scanned in 14.41 seconds

```

ports open : 

-  ftp:21 ---> Vsftpd 3.0.2

- ssh:22 ---> OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)

- http:80 - > Apache and there is website

- rpc:111 *X*



let's navigate to the website first.

![](https://i.imgur.com/4ey9jqg.png)



let's start Fuzzing the web directories. using #ffuf tool.

```bash

ffuf -u http://"IP MACHINE"/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 

```

-w : u can add any wordlist u want. I prefer this one.

![](https://i.imgur.com/Qk5X2bc.png)

We found /island lets navigate into it.

![](https://i.imgur.com/3NUBmLU.png)



we found a Code ==vigilante== . -> **Notice that the code won't display for u until u selected all words**

also u can see the / after the island. it means that there is another directory. subdirectory for island.

let FUZZ this Island directory. using this command

```bash

ffuf -u http://"IP MACHINE"/island/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 

```



and we found ==2100== which is answer for Q2

**Q2**:What is the Web Directory you found?

A2:==2100==



--------------------------------------------

navigate into 2100. 

![](https://i.imgur.com/d3sfdQM.png)

let's try to View the page source. **CTRL+U**

![](https://i.imgur.com/aAOatrm.png)



it seems to be there is .ticket extension exist. and as we can see there is / after 2100. it means as we say before that there is another directory after it. so let's fuzzing it but we will specify to #ffuf   tool that we need .ticket extension. using this command 

```bash

ffuf -u http://10.10.90.135/island/2100/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e .ticket

```

and we got

![](https://i.imgur.com/kvvD03x.png)

==green_arrow.ticket== let's navigate into it

![](https://i.imgur.com/98uAmks.png)

we found this ==RTy8yhBQdscX==

mmm I will use #hash analyzer [URL](https://www.tunnelsup.com/hash-analyzer/)

![](https://i.imgur.com/naFi9IZ.png)



it told us it's base but it's obvious that it's not base64

![](https://i.imgur.com/I4GsxBe.png)

I tried all bases and I found it's base58 it gives me this ==!#th3h00d==

let's try to answer **Q3**. is it ftp password?

![](https://i.imgur.com/qQKIrYF.png)

yes it's..  so let's go join ftp. 

```

ftp "ip machine"

```

asks for username. what if we tried the code that we took in the beginning. ==vigilante==

Username: vigilante , password : !#th3h00d

![](https://i.imgur.com/K48XVRg.png)

see what is it? by using **get**.

![](https://i.imgur.com/t1nkWAa.png)



it's important while u r in ftp. to check for invisible files. **ls -la**

![](https://i.imgur.com/567HfE6.png)

![](https://i.imgur.com/YFdaOGu.png)

we may need the users in this long story.

let's go back for our picture we can see that there is 3 pictures. two of them is .png and one is .jpg ..

jpg can be hide something. png can't.

![](https://i.imgur.com/7K4qfnq.png)



what caused this error? first thing it should came in ur mind that Hex signature is not the appropriate one for png extension.

as we know each extension has their own signature. so what if the signature causing this error? lets find out.

searching in google. **list hex signature** #HexSignature



![](https://i.imgur.com/ExL1vmC.png)



searching for png...

![](https://i.imgur.com/sFgmPLL.png)



==`89 50 4E 47 0D 0A 1A 0A`==

to add this signature we want hex editor. we can found it in kali using this command.

```bash

hexeditor Leave_me_alone.png 

```

![](https://i.imgur.com/oNbOFoZ.png)

as we can see it's not match

so let's edit it manually

Ctrl + x for exit and save

![](https://i.imgur.com/XvRyp5h.png)

great we found "password" but we don't know for what.

it may be a passphrase for jpg picture? it's the only one. let's try it using #steghide using this command

```bash

steghide extract -sf aa.jpg             

```

and it ask for passphrase i put ==password==. and it work we got ss.zip file

![](https://i.imgur.com/4o6K8Z3.png)

extract it. and it gives us two file

![](https://i.imgur.com/3bOjCtg.png)

nothing seems interesting at passwd.txt. check shado.

![](https://i.imgur.com/eAtd3pi.png)

this is what we have got on shado file.

is it a password? let's check the question in tryhackme.

![](https://i.imgur.com/yonw2ry.png)

the answer for it was ==shado== . great it save our time. it is a password for ssh

SSH Password : ==M3tahuman==  , but what is the username. did u remember the long story?

![](https://i.imgur.com/YFdaOGu.png)

it is talking about the man called Slade Willson. what if we try login SSH and trying both Slade and willson

![](https://i.imgur.com/7D3Sk8C.png)

it's correct. the username is Slade. and the password that we found earlier.

the last question asks for root flag. so we need #Linux_privilege_escalation .

check what commands we can run using sudo without password.

![](https://i.imgur.com/Gcf60dP.png)

check our friend #GTFObins 

![](https://i.imgur.com/JlI2yUq.png)



![](https://i.imgur.com/ziFIPss.png)



so we can get root access using this command

```bash

sudo pkexec /bin/sh

```

let's try. and voila we got a root access

![](https://i.imgur.com/ZkzyY9x.png)

