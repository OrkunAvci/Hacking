# Fawn

---

## 10.129.56.217

---

## ping 10.129.56.217

```
PING 10.129.56.217 (10.129.56.217) 56(84) bytes of data.
64 bytes from 10.129.56.217: icmp_seq=1 ttl=63 time=136 ms
64 bytes from 10.129.56.217: icmp_seq=2 ttl=63 time=131 ms
64 bytes from 10.129.56.217: icmp_seq=3 ttl=63 time=99.9 ms
64 bytes from 10.129.56.217: icmp_seq=4 ttl=63 time=151 ms
64 bytes from 10.129.56.217: icmp_seq=5 ttl=63 time=140 ms
^C
--- 10.129.56.217 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss
```

---

## sudo nmap -sV 10.129.56.217 -oX ./nmap/initial

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
```

---

## ftp 10.129.56.217 21

Now this requires some login credentials but we can enumerate them easily. I extracted them out of the walkthrough.

```
Username = anonymous
Password = anon123
```

---

## get flag.txt

and we are done.