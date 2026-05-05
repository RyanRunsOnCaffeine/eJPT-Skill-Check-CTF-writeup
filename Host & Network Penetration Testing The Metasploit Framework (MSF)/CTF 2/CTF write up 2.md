
# Lab Environment

In this lab environment, you will have GUI access to a Kali Linux machine. Two machines are accessible at **target1.ine.local** and **target2.ine.local**.

**Objective:** Using various exploration techniques, complete the following tasks to capture the associated flags:

- **Flag 1:** Enumerate the open port using Metasploit, and inspect the RSYNC banner closely; it might reveal something interesting.
- **Flag 2:** The files on the RSYNC server hold valuable information. Explore the contents to find the flag.
- **Flag 3:** Try exploiting the webapp to gain a shell using Metasploit on target2.ine.local.
- **Flag 4:** Automated tasks can sometimes leave clues. Investigate scheduled jobs or running processes to uncover the hidden flag.

# Tools

The best tools for this lab are:

- Nmap
- Metasploit Framework
- rsync




--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



**Flag 1:** Enumerate the open port using Metasploit, and inspect the RSYNC banner closely; it might reveal something interesting.



First thing is first when it comes to doing any sort of thing on this CTF skill checks. NMAP SCAN!

nmap -sS -sV -O -p- target1.ine.local

So we have 1 open port here which is port 873 which is rsync (protocol version 31)

![](Screenshots%20CTF%202/nmap%20scan%20on%20target1.ine.local.png)


I had never heard of rsync before so this was a new one for me although as I understand it from reading about it online, it's more of a service to synchronise files and directories between two locations. It excels in it's delta-transfer-algorithm which transfers the difference between the source and destination, which saves bandwidth and time making it more efficient.. the more you know!

It mentions in the lab that one of the tools to use is rsync which is a command line utility, so I had a look online and via the help options in linux to get an example command to see how this typically is used to connect to said service. Normally when you connect to things it gives an idea.. this is my thinking anyway going into this.

So I used the command..

rsync rsync://target1.ine.local

Oh hello! Flag 1 has been captured, did not expect that at all! Get it submitted!

![](Screenshots%20CTF%202/Flag%201%20captured.png)








--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------






**Flag 2:** The files on the RSYNC server hold valuable information. Explore the contents to find the flag.

Okay so now we have the first flag, but one thing that stuck out.. what us this "backupwscohen" that it shows there. Can I connect or view that?


rsync rsync://target1.ine.local/backupwscohen/

That's interesting, i'm able to see information on files which will be useful to me. 

![](Screenshots%20CTF%202/checking%20out%20rsync%20backupwscohen.png)



So my train of thought now is how do I get to see this stuff since this must be super important to the tasks, it tell me to check the files as per the flag hints.

So after reading the linux page, -a flag is the archive flag and the -v flag is a verbose flag to see what is happening.. lets give this a whirl and see.

rsync rsync://target1.ine.local/backupwscohen/ .

The above command literally gets these files to my local machine which is excellent and means I can deep dive a bit further into the information which should give me the flag!

![](Screenshots%20CTF%202/rsync%20-a%20-v%20flag.png)


So after checking the files, wouldn't you know it. Flag 2 is hiding in the pii_data.xlsx file! Lets get it captured :D

![](Screenshots%20CTF%202/Flag%202%20captured.png)






--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------








**Flag 3:** Try exploiting the webapp to gain a shell using Metasploit on target2.ine.local.


Okay so we are done with target1.ine.local by the looks of the hint here on flag 3. We now have a new target!

First thing we got to do, is get an nmap scan on this!

nmap -sS -sV -O -p- target2.ine.local

Ahhhhh I see now, we are dealing with an Apache server, versions 2.4.52!

![](Screenshots%20CTF%202/nmap%20scan%20on%20target2.ine.local.png)


I always like to check the website out just to see if I can get a feel of it and what is happening before I plan my next move.

Interestingly, when I viewed the website, an "overview.py" file downloaded straight away.. while at this moment I don't think it means anything its just interesting to know.

![](Screenshots%20CTF%202/viewing%20target2%20website.png)


One thing I noticed between the nmap scan and the actual viewing of the web page is the name "roxy-wi",  this appears to be a web interface for managing servers.. I wonder if it's vulnerable to something.

Searching on metasploit there appears to be an exploit module for Roxy-WI for versions below 6.1.1.0 which is RCE.. Now we are talking

![](Screenshots%20CTF%202/exploit%20roxy-wi%20module.png)

When listing the options, I had changed the following.

set RHOSTS target2.ine.local
set LHOST eth1
run

BOOM. We have a meterpreter session here! Now that we are in, lets have a snoop around!


![](Screenshots%20CTF%202/Meterpreter%20session%20gained.png)


One of the first things I do is always head to the root directory and work my way from there through other directories as you never know what can be hiding.

Luckily enough, the flag is sitting in the root directory just waiting for us! Love it. Lets get Flag 3 submitted!

cd /
ls
cat flag.txt

![](Screenshots%20CTF%202/FLAG%203%20CAPTURED.png)






--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------











**Flag 4:** Automated tasks can sometimes leave clues. Investigate scheduled jobs or running processes to uncover the hidden flag.


Reading the hint.. "scheduled jobs" are known as cron jobs on linux. If it's referring to this which I'm sure it is, we need to find the cron directory. Normally this is configured within the /etc directory if it's a system wide cron jobs. User-specific normally in the var/spool realm of directories.

Using the built in search command in meterpreter, let me search for this.

search -f *cron*

Result. There's a path right there which we will head to right now!

![](Screenshots%20CTF%202/search%20-f%20cron%20in%20meterpreter.png)

Since we got the path, lets head to it,

cd /etc/cron.d
cat www-data-cron

Boom. There's yer dinner. Flag 4 in the bag. Submit this and complete CTF 2! :D


![](Screenshots%20CTF%202/flag%204%20captured.png)



--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



All flags submitted!


![](Screenshots%20CTF%202/all%20flags%20captured.png)



Overall this CTF was decent, Using and figuring the rsync commands was a bit of a learning curve since I hadn't used it before but once I got an example command to connect to it. It kinda twigged in my head how this would work!

The other thing that I noticed that got me a little bit was not search for "roxy-wi" on metasploit, I kept searching for apache but out of randomness I thought to myself, if i search for this its worth a shot and luckily enough I did. I need to keep stuff like that in my head for future :)
