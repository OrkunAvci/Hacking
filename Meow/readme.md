# Meow

---

## 10.129.56.191

---

## sudo nmap -sV 10.129.56.191 -oX ./nmap/initial

```
PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
```

---

## telnet 10.129.56.191

Only requires username and we try out the admin usernames.

Parallel with the walkthrough, `admin`, `adminisrator`, and similar is incorrect.

The correct one is `root`.

Now we have root access and can grab the flag.