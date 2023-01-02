# Active directory Machine

First of all, we gonna make a port scan using nmap:

    nmap <target_ip>
   
![enter image description here](https://i.ibb.co/BKXw0nG/Screen-Shot-2022-12-30-at-8-01-00-PM.png)

Since we have credentials from the lab description, we gonna try to connect using rpcclient:

    rpcclient -U [DOMAIN/]USERNAME[%PASSWORD] <target_ip>

![enter image description here](https://i.ibb.co/30fPDsZ/Screen-Shot-2022-12-30-at-8-06-05-PM.png)

Now we can enumerate some information, to see if we got anything.
This is an article that could help you: [Rpcclient](https://www.hackingarticles.in/active-directory-enumeration-rpcclient/)
We can query all account with their information using ```querydispinfo```

![enter image description here](https://i.ibb.co/Kz6YF2P/Screen-Shot-2022-12-30-at-8-09-35-PM.png)

And there it is in-front of our eyes the password of test_av account in its description.
We can verify that test_av is an Administrator:

![enter image description here](https://i.ibb.co/51KLkSK/Screen-Shot-2022-12-30-at-8-22-31-PM.png)

We use ```enumdomgroups``` to list domain groups, we can see the RID of Domain Admins group, now we should ```enumdomusers``` to list users and there we find the RID of test_av users.
Now to check if test_av is an Administrator we can enumerate user groups using ```queryusergroups``` and give it the RID of the user, and there it is, test_av is an Administrator.

Now we gonna try to change the password of test_av using ```chgpasswd```

    chgpasswd test_av antivirus123! antivirus123@

Now we can connect using ```rdesktop``` with the username test_av and the new password, it will redirect us to change the password again choose a new password, and there we are we got access to the machine, run cmd.exe as an Administrator, and search for the flag.
