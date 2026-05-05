
Inital starting the lab:

![](Screenshots/Starting%20the%20lab.png)

Website as shown - http://target.ine.local

![](Screenshots/target.ine.local%20website.png)

- **Flag 1**: The server proudly announces its identity in every response. Look closely; you might find something unusual.



command used:

whatweb target.ine.local

FLAG1_48b3b9369b504ed9b1fc56cbf8f38e16

![](Screenshots/Flag%201%20-%20whatweb%20command.png)

- **Flag 2**: The gatekeeper's instructions often reveal what should remain unseen. Don't forget to read between the lines.

Command used:

curl -i http://target.ine.local/secret-info/flag.txt

FLAG2_51ac7dc7ac2b41b099ac7c048361ef2a

![](Screenshots/robots.txt%20target.ine.local.png)
![](Screenshots/secret-info%20target.ine.local.png)

![](Screenshots/flag.txt%20target.ine.local.png)

![](Screenshots/curl%20target.ine.local%20to%20flag.txt.png)


- **Flag 3**: Anonymous access sometimes leads to forgotten treasures. Connect and explore the directory; you might stumble upon something valuable.

when I heard directory I automatically thought of directory busting but re-reading this, it gave me a clue, anonymous & connect..

I remember there was a demo at the end of the demo where the instructor connect to tftp server.. must be the case here however. I wanted to find out more information, since this was footprinting & scanning, I must use nmap!

command used:

nmap -sS -p- target.ine.local
nmap -sS -A -T4 -p- target.ine.local

ftp target.ine.local (name - anonymous, password - "hit enter")
ftp> ls
ftp> get creds.txt
ftp> get flag.txt

![](Screenshots/Basic%20all%20ports%20SYN%20scan%20nmap.png)

![](Screenshots/Fuller%20nmap%20scan%20to%20show%20FTP%20info.png)

![](Screenshots/ftp%20login%20successful.png)

![](Screenshots/ftp%20ls%20.txt%20files.png)

![](Screenshots/ftp%20file%20transfer%20complete.png)

exit, get ouuta here!

FLAG3_90fd9065ee6d43acb5062bd3818bda57

![](Screenshots/flag%203%20flag.txt%20file%20opened.png)

Being the sneaky sneaky I am, lets have a peak at these credentials..

db_admin...... this must be the database credentials!

![](Screenshots/creds.txt%20opened.png)







- **Flag 4**: A well-named database can be quite revealing. Peek at the configurations to discover the hidden treasure.

commands used:

nmap -sS -A -T4 -p3306,33060 target.ine.local

mysql -u db_admin -p -h target.ine.local (username - db_admin, password - password@123)

mysql> showdatabases;


![](Screenshots/mysql%20namp%20scan.png)


![](Screenshots/connected%20to%20mysql%20commands.png)

![](Screenshots/show%20databases;%20mysql%20for%20flag%204.png)





CTF finished:

![[CTF finished.png]]