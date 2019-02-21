# MR Robot writeup 

## Recon

IP found using netdiscover 

Wordpress installation found using dirb (http://<IP>/0)
has XMLRPC enabled. this allows us to send batch password requests.
dirb also found robots.txt with contains

```bash
User-agent: *
fsocity.dic
key-1-of-3.txt
```

seems like http://<IP>/key-1-of-3.txt gives the first key

```bash
073403c8a58a1f80d943455fb30724b9
```

it also shows a file that contains a bunch of words (a wordlist).
 
## Wordpress login

wpscan can give us some more information about the website
by running `wpscan --url http://<ip>/0 --wp-content-dir /wp-content --enumerate u` we can show what users exist.

```bash
[i] User(s) Identified:

[+] mich05654
 | Detected By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)


[+] elliot
 | Detected By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)

```

this allows us to create a userlist to login with.
by using the XMLRPC.php exploit we can easilly try the large fsocity.dic wordlist to login

login credentials:

```bash
[SUCCESS] - mich05654 / Dylan_2791  
[SUCCESS] - elliot  / ER28-0652   
```

mich is just a user but elliot is an administrator.


## Wordpress reverse php shell

By checking plugins we can see an import and export plugin.
this might give us an easy way of uploading some php files.
by adding our backdoor to the exported zip we can get a reverse shell.

## Privilege escalation

Now that we can run commands we can see what's on the system.
we can see that there is a /home/robot directory containing

```bash
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
```

this is where the second key can be found.
This is where i might deviate from other writeups.
I did not yet know a way of spawning a tty (using python for example)

## Getting root

By checking for files which have their suid bit set we can find nmap.
nmap contains an interactive mode that can spawn a shell

```bash
/usr/local/bin/nmap --interactive
nmap> !sh
whoami
root
```

## Succes!

This allows us to both read key 2 and key 3 (which can be found in /root)

```bash
key2: 822c73956184f694993bede3eb39f959
key3: 04787ddef27c3dee1ee161b21670b4e4
```

with this i've finished the MrRobot vulnhub machine