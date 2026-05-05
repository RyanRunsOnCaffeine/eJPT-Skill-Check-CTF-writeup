
# Lab Environment

A Linux machine is accessible at **target.ine.local**. Identify the services running on the machine and capture the flags. The flag is an md5 hash format.

- **Flag 1:** There is a samba share that allows anonymous access. Wonder what's in there!
- **Flag 2:** One of the samba users have a bad password. Their private share with the same name as their username is at risk!
- **Flag 3:** Follow the hint given in the previous flag to uncover this one.
- **Flag 4:** This is a warning meant to deter unauthorized users from logging in.

**Note:** The wordlists located in the following directory will be useful:

- /root/Desktop/wordlists

# Tools (suggested)

- Nmap
- Metasploit
- Hydra
- enum4linux
- smbclient
- smbmap



Flag 1: There is a samba share that allows anonymous access. Wonder what's in there!

So interestingly a directory was given to us /root/Desktop/wordlists

out of curiosity I wanted to see what was in there as this could leave big clues for future flags...


I changed to directory and say shares.txt and unix_passwords.txt

unix_passwords.txt will more than likely be used for brute forcing a little later but whats the shares one...

![](Screenshots/cat%20shares.txt.png)

it has a list of potential shares on the SMB which may give us an indicator on how to get the shares..


I ran enum4linux to see if I could enumerate the shares to find out which shares allows anonymous login.. the condensed interesting results were as follows:

command:
enum4linux -a target.ine.local

![](Screenshots/enum4linux%20screenshot%201.png)
![](Screenshots/enum4linux%20screenshot%202.png)

So I can see print$ and IPC$ shares so far, a quick google search shows that print$ is a share for print drivers and IPC$ is used for communication between network programs, I connected to them but nothing for found within those shares.

What's interesting is this has also gave me some users to keep in mind.. josh, bob, nancy, alice, nobody

Let's keep that in mind as this may come in handy later.

Still no sign of the anonymous login for the flag, I was scratching my head until I remembered again about the file named shares.txt as this was a hint to get me thinking, if a share was to allow access anonymously, what would it be named? My first instinct was to try ones that had public in it.

I connected to each manually using anonymous logins to observe the behaviour until bingo.. I found something!

![](Screenshots/smbclient%20connect%20anon%20flag.png)

lets quickly get that flag downloaded and shown so we can submit it!

![](Screenshots/flag%201%20captured.png)






----FLAG 2-------





------------------------------------------------------------------------
Flag 2:  One of the samba users have a bad password. Their private share with the same name as their username is at risk!

first thing is to run an nmap scan to gain an understanding of what is happening  on the target on all tcp ports.

Good results shown from nmap showing the version of Samba and the relevant ports, exactly what we are looking for.. However port 5554 open for FTP? I'll investigate that later but for now, the samba information and version is gucci

![](Screenshots/nmap%20all%20TCP%20ports%20scan.png)

Let's launch Metasploit framework and get to work on finding out a bit more on Samba

Firslty, setting setting global variables to avoid more work

![](Screenshots/MSF%20global%20variables.png)

Now that global variable are set up. Let's look at enumerating samba as per the questions.

Firstly, I want to see what type of auxiliary modules we have here to really understand what we are going to do first, normally service version even though it's not asking for it and nmap diod provide it, doesn't hurt to double check. (You never know!)

I used the following command to list down the modules:

search type:auxiliary name:smb

We are looking for the module which will tell us the smb version, as can see here it is at number 38.

auxiliary/scanner/smb/smb_version

Lets use this module.

![](Screenshots/SMB%20version%20module.png)


listing the show options command, this will tell us what is required, already set up global varibale for RHOSTS, dont need to specify RPORT unless scans tell us different port used.

Lets run.

![](Screenshots/smb%20version%20module%20run.png)

as I mentioned it's good practice to check the information just to keep yourself right.


Okay lets start enumerating some users. I mentioned in flag 1 section we had some users which was shown via enum4linux (josh, bob, nancy, alice, nobody).

The module used here is auxiliary/scanner/smb/smb_enumusers, once set, it was run.

![](Screenshots/smb%20enumusers%20result%20MSF.png)


Fantastic! We got josh, nancy and bob. This looks to be some of the accounts we got earlier!



Now let's just to the brute force module to see if we can get more information from this. There's 2 attempts I want to run here, via a wordlist and then by the accounts josh, nancy, bob that we have just found. 

The module we want to use in this case is auxiliary/scanner/smb/smb_login, and configure the options.
auxiliary/scanner/smb/smb_login - set user file and pass file, show get 7 including admin etc

set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /root/Desktop/wordlists/unix_passwords.txt
set VERBOSE false

Lets run the module.

![](Screenshots/smb%20brute%20force%20with%20lists%20output.png)


Oh look at that, some very important logins! sysadmin, rooty etc with the password admin now THAT is a jackpot for any penetration tester.


The flag hint says that one of the samba users has a bad password and the accounts I found just there really aren't the users I believe I should be looking for.. what about josh, nancy and bob? What about alice and nobody?

Let's add them to a file called smbusers.txt, each in a seperate line for our msf to read in the file.

![](Screenshots/smb%20users%20added%20to%20a%20file.png)

This is saved in the /root directory which is important to specify when we are setting the USER_FILE variable in MSF.

Okay, let's change the USER_FILE in MSF now to point to the smbuser.txt file I made.

set USER_FILE /root/smbusers.txt

Note: the option to singularly to set SMBUser (username) is also possible however I wanted to be thorough with the information provided to maximise completeness.

anddddd run

![](Screenshots/smb%20brute%20force%20with%20smbusers.txt.png)

Josh:purple .. now that's different and interesting. josh must be the user in question the flag is talking about!!




Lets connect to smbclient with the josh username and josh as the share.

bingo, theres flag2.txt right there!

lets GET the file and open it up on our local machine.

Flag captured, inputted and in the bag :)

![](Screenshots/Flag%202%20caught.png)

Psst! I heard there is an FTP service running. Find it and check the banner.








-----Flag 3

----------------------------------------------------------------------

Flag 3:  Follow the hint given in the previous flag to uncover this one.

Now, that is interesting, remember earlier I ran the nmap scan and commented to why FTP was running on another port than the typical port 21? This now makes 100% sense to why it was there. I guess I kinda ruined the fun early on without knowing it (or this might be future me writing this up already knowing the surprise who knows)

Lets use another module relating to ftp now, lets search the list of available modules that we can use..

search type:auxiliary type:ftp

The module used will be auxiliary/scanner/ftp/ftp_version since we are looking to see the banner.

![](Screenshots/ftp%20service%20version%20MSF.png)

Important part here is when we set the options for this module, it auto configures to RPORT 21 as this is the typical port used by ftp HOWEVER sneakily it has changed and been configured to port 5554. Let's set that.

set RPORT 5554

and run the aux module..

![](Screenshots/FTP%20banner.png)

huh interesting. "[+] 192.54.22.3:5554      - FTP Banner: '220 Welcome to blah FTP service. Reminder to users, specifically ashley, alice and amanda to change their weak passwords immediately!!!\x0d\x0a'"

Users ashley, alice and amanda are now in the firing line.


What's interesting is the tools listed for the CTF as suggestions shows hydra, this is a brute force password tool which we can use especially for services such as SSH, FTP etc. We could use an auxiliary module for ftp_login however in the spirit of changing it up, lets do this and gain exposure to other tools.

I now want to do something similar to the SMB and add these 3 users to their own specified .txt file because I already know that I can either specify a username or a wordlist.

Same as before adding ashley, alice, amanda to a .txt file called ftpusers.txt.

![](Screenshots/adding%20ftp%20users%20to%20.txt%20file.png)


I am quite familiar with hydra as I have used it previously in different labs however using the command "hydra -h" brings the help menu to show how to use the tool, if you scroll at the bottom there is an example command on how to use the tool and luckily enough, one of the commands is close to what we will be using.

![](Screenshots/hydra%20-h%20example.png)

lets get bruteforcing!

command:

hydra -L ftpusers.txt -P /root/Desktop/unix_passwords.txt ftp://target.ine.local:5554

![](Screenshots/hydra%20brute%20force%20success.png)

And there we have it! Target found! login: alice password: pretty

Let's log into this ftp client using those credentials...

command:

ftp -p target.ine.local 5554

Let's go, we are now connected!!

![](Screenshots/ftp%20login%20as%20aslice%20works.png)

Let's have a nosey to see what stuff is here and what would you know, flag3.txt just sitting there ready for the taking, lets get this file and open the file on our local machine so we can submit flag 3!

Flag 3 submitted and done!

![](Screenshots/flag%203%20captured.png)







-------------- FLAG 4


----------------------------------------------------------------------

Flag 4: This is a warning meant to deter unauthorized users from logging in.

Firstly I'm thinking back to the nmap scan we done earlier, as surely there's got to me pieces to the puzzle.

We have done samba, we have located the sneaky FTP on a different port, the other service left is SSH.

Now at the time of doing these labs I'm a 3rd year cyber security student at university, I'm SSH'd out my eyeballs with the amount of labs I've done. (Thinking of my cryptography exam lab setting up PKI infrastructures between client's and servers, transferring certificates & files between multiple servers *Shivers*)

lets simply try the command:

ssh target.ine.local

and there it is, flag4 just sitting there ready for the taking!


![](Screenshots/FLAG%204%20captured.png)

Overall summary:

Very good CTF, I did scratch my head for a bit to understand what I was looking for and the throw in for hydra was nice.