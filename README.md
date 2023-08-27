# Mr-Robot-CTF
A step by step walk through for the Mr robot CTF on Tryhackme

---

Link: https://tryhackme.com/room/mrrobot

---

Lets begin with nmap, a popular enumeration tool!

more here: https://www.kali.org/tools/nmap/

Command: nmap <machine_ip> -sV -sC -p- (We use -sV to determine the service version of the services running on the open ports and we use -p- to scan all ports, because by default nmap only scans 1000 ports, we can change it to scan all 65,000 using -p-)

Anways! Lets look at our results

![image](https://github.com/traveller404/Mr-Robot-CTF/assets/92340426/3a9b8008-9e0f-4d6b-b73e-8193011e6a9d)

We see that port 22, 80 and 443 are open ,these are quite common, given that 80 and 443 are website ports, lets go to the website using the url *http://<machine_ip>*

![image](https://github.com/traveller404/Mr-Robot-CTF/assets/92340426/e1a6d467-3024-45d5-96fe-331195b71f5f)

Inputting any of the availabe commands returns nothing useful, and checking the source code doesnt either, but I still recommend you play around with it because it is a really entertaining machine.

Going to a common subdomain of */robots.txt* reveals 2 more subdomains!

![image](https://github.com/traveller404/Mr-Robot-CTF/assets/92340426/ac3195a2-6f0c-4dbb-baa0-8d1ccd7fe142)

Going to */key-1-of-3.txt will give you one of the 3 flags we retrieve through out this CTF, and going to *fsociety.dic* will download a long list of certain words and phrases

With nowhere else to go, we can now try directory enumeration through the use of Gobuster

Command: `gobuster dir -u http://<machine_ip> -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt`

![image](https://github.com/traveller404/Mr-Robot-CTF/assets/92340426/ea12b0c2-c990-4675-9f7f-ec94c7f2f31c)

After a while we are left with a subdomain of /admin, this brings us to a reload loop, and another subdomai of /wp-admin.php which we can login to. (*WordPress if you dont know is a website management system, used by many websites accross the internet*)

# Brute forcing the login page

We now have a long collection of words and strings in the fsocity.dic file, but first we need to configure that

![image](https://github.com/traveller404/Mr-Robot-CTF/assets/92340426/83fe59e3-db37-49f4-87a8-cbf02fd773e5)

Seeing that there is alot of words in this file, this isnt mandatory but were going to check if there are any duplicate words in this file

![image](https://github.com/traveller404/Mr-Robot-CTF/assets/92340426/082d00b4-747c-4480-828e-2b482c77fa58)

![image](https://github.com/traveller404/Mr-Robot-CTF/assets/92340426/57338630-d17a-4b60-9c2f-cf39cc3a9429)

What I did is i used `wc -l fsocity.dic` and it told me there was 80,000+ words
I then used `sort fsocity.dic | uniq > filtered.dic` and what this does is that it sorted the fsocity file, and I took all the uniq words and put it into a file called filtered.dic, but in your case you can name it anything you want

Time to write our hydra command     *more on hydra here: https://www.kali.org/tools/hydra/*

(if you are confused about hyda, and burpsuite I reccomend trying out these tryhackme rooms https://tryhackme.com/room/hydra and https://tryhackme.com/module/learn-burp-suite)

First we have to get our burpsuite fired up, to intercept our browser traffic and see what variables in the website store our username and password credentials

![image](https://github.com/traveller404/Mr-Robot-CTF/assets/92340426/24e1cf84-b7e5-42d9-b279-7b08bce95f03)

We see that the username is stored as "log" and the password as "pwd" and at the top it says POST, for http-post-form

Command: `hydra -L filtered.dic -p `test (this can be anything)` 10.10.40.121 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+in:F=Invalid username." -V` *-V is optional, and you only use it if you wanna feel like your out of a movie*

Once leaving this running ,we get a valid username for the webpage, and we now can change our hydra command to fit this, and get us a password

If everything has ran well you should get now have a password aswell, in this point of the CTF we have access to upload a reverse shell using Wordpress's plugin feature

# Reverse Shell

The appearance section of the wp-admin we can edit any of the files to be a reverse shell to our own machine, you can find a php reverse shell here https://github.com/pentestmonkey/php-reverse-shell

Change any files in this area, I have chosen archive.php and changed the contents of the reverse shell to my IP address and set up a nc listener on port 9001 using `nc -lvp 9001` and changed the reverse shell file to match that too

Then going to http://<machine_ip>/wp-content/themes/twentyfifteen/archive.php, checking back on my nc listener I now have a reverse shell

# Privelege Escalation

With the reverse shell we now have we can now find our 2nd and 3rd flag

Going to /home/robot, we see the 2nd flag and another file, containing a password for another user called robot, but it is hash encrypted, going to https://crackstation.net and inputting this 
we get a cracked md5 hash and a password

There is a problem though, trying to su to robot requires us to be in a proper terminal instead of a reverse shell, but this can be fixed easily by using `python -c 'import pty; pty.spawn("/bin/sh")'` 

# Privelege Escalation to root

Now that we have some permissions, we can run a command that searches for files that have root SUID permissions

`find / pern +6000 2>/dev/null | grep /bin`

From this we find that nmap is returned among many others

This can be used to gain root access to the machine, as nmap has an interactive mode that can be used to execute commands

`nmap --interactive`

followed by

`!sh`

Change to /root and get the 3rd flag

Hope this helps!
