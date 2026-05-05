### Completing Skill Check Labs

Skill Check Labs are interactive, hands-on exercises designed to validate the knowledge and skills you’ve gained in this course through real-world scenarios. Each lab presents practical tasks that require you to apply what you’ve learned. Unlike other INE labs, solutions are not provided, challenging you to demonstrate your understanding and problem-solving abilities. Your performance is graded, allowing you to track progress and measure skill growth over time.

# Lab Environment

In this lab environment, you will be provided with GUI access to a Kali Linux machine. Two machines are accessible at **http://target1.ine.local** and **http://target2.ine.local**.

**Objective:** Perform system/host-based attacks on the target and capture all the flags hidden within the environment.

**Useful files:**

```
/usr/share/metasploit-framework/data/wordlists/common_users.txt, 
/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt,
/usr/share/webshells/asp/webshell.asp
```

**Flags to Capture:**

- **Flag 1**: User 'bob' might not have chosen a strong password. Try common passwords to gain access to the server where the flag is located. (target1.ine.local)
- **Flag 2**: Valuable files are often on the C: drive. Explore it thoroughly. (target1.ine.local)
- **Flag 3**: By attempting to guess SMB user credentials, you may uncover important information that could lead you to the next flag. (target2.ine.local)
- **Flag 4**: The Desktop directory might have what you're looking for. Enumerate its contents. (target2.ine.local)

# Tools

The best tools for this lab are:

- Nmap
- Hydra
- Cadaver
- Metasploit Framework





**Flag 1**: User 'bob' might not have chosen a strong password. Try common passwords to gain access to the server where the flag is located. (target1.ine.local)


First thing is always to run an nmap scan just to be sure, getting the information on the target is really important as it gives us accurate technical data on what we are working with here.

nmap -sSVC -p- target1.ine.local

(nmap scan results here)


Let's have a look at the website to see what it's saying.

Typing this straight away, it's asking for a username and password. Regardless of where I try to go directory wise, it still asks for the credentials.

![](Screenshots/target1.ine.local%20Website%20first%20look.png)

So the hint was bob as the username, lets brute force it was hydrA

Password has been found for bob.. his password is password_123321


![](Screenshots/brute%20force%20password%20found%20for%20bob.png)


Typing in the credentials we brute forced, we are now in to the website!


![](Screenshots/target1.ine.local%20logged%20in!.png)



So not much information given so far, the nmap scan showed a IIS on the web server, I wonder if it has a /webdav directory..

and look at that, there's the first flag just waiting for me! Let's get it submitted!


![](Screenshots/checking%20the%20webdav%20directory.png)

![](Screenshots/flag%201%20captured!.png)












**Flag 2**: Valuable files are often on the C: drive. Explore it thoroughly. (target1.ine.local)

One thing I noticed after checking the /webdav directory is there is a test.asp file on there, clicked on it and it ran.. this means this web directory allows these files to be executed on there.. lets get a webshell in there since it's asking to explore the C: drive

(In a real scenario, if the .asp file wasn't already there, theres a davtest tool I could have utilised)

Using the cadaver tool, I am going to connect to it so I can look at getting a webshell on there :)

P.S.. Flag1 could also be captured via this way :)

![](Screenshots/cadaver%20tool%20logged%20in%20with%20creds.png)

![](Screenshots/AF1%20capture.png)


To upload the webshell, there was another absolute path given to us in the overall briefing of the CTF.

/usr/share/webshells/asp/webshell.asp

lets get that up on the web server in the /webdav directory so we can get remote code execution!

put /usr/share/webshells/asp/webshell.asp


![](Screenshots/web%20shell%20uploaded%20via%20cadaver.png)


And just to confirm this has landed, let's check the /webdav directory...


![](Screenshots/webshell%20in%20webdav%20directory.png)


We now officially have remote code execution on the target via the webshell.asp payload we uploaded on to the target site!


![](Screenshots/RCE%20on%20web%20server.png)



So lets now run the following commands:

dir c:\
type c:\flag2.txt

There is flag2.txt right there in the C drive just as expected. Let's get it open and submitted!



![](Screenshots/dir%20c%20command%20in%20RCE.png)


![](Screenshots/FLAG%202%20FOUND.png)












**Flag 3**: By attempting to guess SMB user credentials, you may uncover important information that could lead you to the next flag. (target2.ine.local)

Also a further clue (SMB shares might contain hidden files. Check the available shares.)


Attempting to guess SMB user credentials? I will keep that in mind., lets run an nmap scan first.


![](Screenshots/nmap%20scan%20target2.ine.local.png)



Now I'm going to try and see if I can do some sort of brute forcing of SMB credentials to see if I can get access.

Lets load up msfconsole and have a look at the auxiliary scanners for this.

The selected module is "auxiliary/scanner/smb/smb_login"

set the USER_FILE to the one stated in the briefing. (common_users.txt)
set the PASS_FILE to the one stated in the briefing (unix_passwords.txt)
set RHOSTS target2.ine.local

I also turned Verbose to false as I didn't want a full screen lol..

Interesting, we have now found some users & password via bruteforcing.. including an Administrator.

administrator:pineapple

(Note: Decided to opt for metasploit on this however hydra is typically the go to for this, the command used for hydra would be "hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt smb://target2.ine.local")


![](Screenshots/SMB%20brute%20force%20target2.png)

Lets now check some of the shares that are available with our new shiny creds!

using the "auxiliary/scanner/smb/smb_enumshares" module.

set RHOSTS target2.ine.local
set SMBUser Administrator
set SMBPass pineapple
run

ohhhhh look at the shares we have got :D


![](Screenshots/SMB%20shares%20found.png)


Now to connect to these shares and find out what's hiding underneath :)

After trying ADMIN$ share to no avail, the next one is C$..

smbclient //target2.ine.local/C$ -U administrator

Bingo, theres flag3.txt.

As we are in the smb session, lets get the file and open the contents.

get flag3.txt
(exit smb session)

cat flag3.txt

CAPTURED. LETS GOOOO!


![](Screenshots/Flag%203%20found!.png)


![](Screenshots/flag%203%20captured!.png)










**Flag 4**: The Desktop directory might have what you're looking for. Enumerate its contents. (target2.ine.local)


Now this one is easier since it tells us to go to the Desktop directory in here..

lets reconnect the session (since I got excited and exited when we captured flag3.txt)

If you see from the last screenshot within flag 3, it lists out the contents and directories. Now I know already that we need to head into the Users directory as typically we need to select the user and then into the desktop directory.

Let's do these commands:

cd Users
cd Administrator
cd Desktop

Boom, there's flag4.txt just sitting, get this downloaded and opened up for the big W!

Downloading the file using:

get flag4.txt

exiting the session and then opening the file via cat, get that slammed in for the submission!


![](Screenshots/flag%204%20captured.png)






ALL SUBMITTED!


![](Screenshots/All%20flags%20submitted.png)