# SEED LAB: FIREWALL Report

## 0. Configuration
In this report, we used two virtual machines and set the all in same NAT networks. We also used the commands below to configure their bash prompt, which can help us identify them more easily. 

```Shell
# Commands for the victim 01
$ echo 'export PS1="\[\e[1;34m\][A]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
# Commands for the victim 02
$ echo 'export PS1="\[\e[1;32m\][B]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
```

As the image shows, we used the left as the VM "<ins style="color:red;font-weight:bold">A</ins>" (`10.0.2.15`) and the right one as the VM "<ins style="color:blue;font-weight:bold">B</ins>" (`10.0.2.6`). 

<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_25932abafd1c95e825a5beb6959e5ea4.png"/>
<figcaption>Figure 0.1 "Bash Promp" for the two VMs</figcaption><br>
</figure>

<div style="page-break-after: always;"></div>

## 1. TASK01: Using Firewall

### 1.1 Preparing for iptable

Before starting, we executed the two commands below to disable `ufw` and flush all current iptables rules.

<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_5086bd89b2d956c0b684238d5fe037f6.png"/>
<figcaption>Figure 1.1.1 commands executed before the task01</figcaption><br>
</figure>

### 1.2 Prevent A from doing telnet to Machine B

As image below shows, the `telnet` command worked between the "<ins style="color:red;font-weight:bold">A</ins>" and "<ins style="color:blue;font-weight:bold">B</ins>" before the configuration of our firewall. 

<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_7013e5fb4fc33b447161ae371c1efba0.png"/>
<figcaption>Figure 1.2.1 `telnet` between the two VMs</figcaption><br>
</figure>

We used the command below to configure the firewall on "<ins style="color:red;font-weight:bold">A</ins>" to filter out all package to IP `10.0.2.6` ("<ins style="color:blue;font-weight:bold">B</ins>"). Detaily, the command added a rule to the table **FILTER** and the chain **OUTPUT**. We specified all packages to **dest IP** `10.0.2.6` used **protocol** `tcp` and **dest Port** `23` will be dropped (`DROP`).  

```shell
sudo iptables -t filter -A OUTPUT \
    -d 10.0.2.6 -p tcp --dport 23 \
    -j DROP
```

The output of `iptables -L` below shown that the rule has already been accepted by the firewall on "<ins style="color:red;font-weight:bold">A</ins>". As the image shows, we also executed the `telnet` again, and it did not work this time.

<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_db9be197b0d5afbd6e3517a43fc98b96.png"/>
<figcaption>Figure 1.2.2 The commands used in the task01</figcaption><br>
</figure>

### 1.3 Prevent B from doing telnet to Machine A

The image below shows that the `telnet` command worked on "<ins style="color:blue;font-weight:bold">B</ins>" to "<ins style="color:red;font-weight:bold">A</ins>". The 

<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_7b1deaa75830693b128f7972628b57dd.png"/>
<figcaption>Figure 1.3.1 Telnet worked before configuring</figcaption><br>
</figure>

We modified the previous command to below one, which applied rule on firewall of "<ins style="color:red;font-weight:bold">A</ins>" to filter out all package from IP `10.0.2.6` ("<ins style="color:blue;font-weight:bold">B</ins>"). Compared with previous, we used the chain to **INPUT** and we limited the **src IP** (option: -s) instead of **dest IP** (option: -p). 

```shell
sudo iptables -t filter -A INPUT \
    -s 10.0.2.6 -p tcp --dport 23 \
    -j DROP
```

The image below shows that the same command in Figure 1.3.1 timed out after configuring. 

<figure>
<img style="width:100%; padding-left:0%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_b6cc32d35f98993110b5490be869682d.png"/>
<figcaption>Figure 1.3.2 Telnet timed out after configuring</figcaption><br>
</figure>

### 1.4 Prevent A from visiting an external web site

In this subtask, we went to stop the package to the official website of JHU `avirubin.com`. We firstly used the command `nmap` to obtain the ip address and the open port number of website. We image below shows its output, in which the IP is `18.220.192.230` and three port number  are avaliable (SSH`22`, HTTP`80`, HTTPS`443`)

<figure>
<img style="width:60%; padding-left:15%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_469805aa37edcc64c490e93b4c7e5f47.png"/>
<figcaption>Figure 1.4.1 Result for nmap scanning to avirubin.com</figcaption><br>
</figure>

After that, we used the command `curl -I` to verify whether the request is dropped. The option `-I` there means only return the header because we only care about whether the request overtimed or not. As the image below shows, the request to the url worked well before configuring. 

<figure>
<img style="width:50%; padding-left:25%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_88472e50edfc885f4f0f7657467c11cc.png"/>
<figcaption>Figure 1.4.2 Request to avirubin.com before configuring</figcaption><br>
</figure>

And then, we used the command, `sudo iptables -t filter -A OUTPUT -d 18.216.86.163 -p tcp --dport 80 -j DROP`, to configure the firewall on "<ins style="color:red;font-weight:bold">A</ins>". After that, we found the `curl -I` command overtimed. 

<figure>
<img style="width:100%; padding-left:0%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_aef85d68350bf892ca94b522f2504aff.png"/>
<figcaption>Figure 1.4.3 Request to avirubin.com after configuring</figcaption><br>
</figure>

<div style="page-break-after: always;"></div>

## 2. TASK02: Implementing a Simple Firewall

Before begining the task, we cleared the `iptables` firewall rules we set in the previous task running the command `sudo iptables -F`.

We wrote the below c program to a simple implement packet filtering module using `Netfiler`. Our program implements the following rules. Our first three rules were binded with NF_INET_LOCAL_OUT hook, which will handle all outgoing packages.  
* **RULE01**: Block telnet to Machine "<ins style="color:blue;font-weight:bold">B</ins>" (Stop outgoing packages with dst port 23 and dst IP 10.0.2.6)
* **RULE02**: Block http to `avirubin.com` (Stop outgoing packages with dst port 80 and dst IP 10.0.2.6)
* **RULE03**: Block https to `avirubin.com` (Stop outgoing packages with dst port 443 and dst IP 10.0.2.6)

Our lasr two rules were binded with NF_INET_LOCAL_IN hook, which will handle all incoming packages(Does not include the forwarding one).  
* **RULE04**: Block telnet from Machine "<ins style="color:blue;font-weight:bold">B</ins>" (Stop incoming packages with dst port 23 and src IP 10.0.2.6)
* **RULE05**: Block ssh from Machine "<ins style="color:blue;font-weight:bold">B</ins>" (Stop incoming packages with dst port 22 and src IP 10.0.2.6)


```c
static struct nf_hook_ops out_filter_hook;
static struct nf_hook_ops in_filter_hook;

unsigned int local_out_hook(
  void * priv, struct sk_buff * skb, 
  const struct nf_hook_state * state
){
  struct iphdr * ip_header = (struct iphdr *)skb_network_header(skb);
  struct tcphdr * tcp_header = (struct tcphdr * )skb_transport_header(skb);
  unsigned int ip_dst_addr = (unsigned int)ip_header->daddr; 

  if (ip_header->protocol != IPPROTO_TCP) return NF_ACCEPT; 
  /* RULE01: block telnet to Machine B */
  if (tcp_header->dest == htons(23) // port number 23; 
    && ip_dst_addr == htonl(0x0a000206)){ //dst ip 10.0.2.6; 
    return NF_DROP;
  }
  /* RULE02: block http to avirubin.com */
  if (tcp_header->dest == htons(80) // port number 80; 
    && ip_dst_addr == htonl(0x12d856a3)){ //dst ip 18.216.86.163 ; 
    return NF_DROP;
  }
  /* RULE03: block https to avirubin.com */
  if (tcp_header->dest == htons(443) // port number 443; 
    && ip_dst_addr == htonl(0x12d856a3)){ //dst ip 18.216.86.163 ; 
    return NF_DROP;
  }
  return NF_ACCEPT; 
}

unsigned int local_in_hook(
  void * priv, struct sk_buff * skb, 
  const struct nf_hook_state * state
){
  struct iphdr * ip_header = (struct iphdr *) skb_network_header(skb);
  struct tcphdr * tcp_header = (struct tcphdr *) skb_transport_header(skb);
  unsigned int ip_src_addr = (unsigned int) ip_header->saddr; 

  if (ip_header->protocol != IPPROTO_TCP) return NF_ACCEPT; 

  /* RULE04: block telnet from Machine B */
  if (tcp_header->dest == htons(23)//portnumber cannot be 23 (Telnet); 
    && ip_src_addr == htonl(0x0a000206)){ //dst ip 10.0.2.6; 
    return NF_DROP;
  }
  /* RULE05: block ssh from Machine B */
  if (tcp_header->dest == htons(22)//portnumber cannot be 22 (Telnet); 
    && ip_src_addr == htonl(0x0a000206)){ //dst ip 10.0.2.6; 
    return NF_DROP;
  }
  return NF_ACCEPT; 
}

int init_module(void)
{ 
  out_filter_hook.hook = local_out_hook;
  out_filter_hook.hooknum = NF_INET_LOCAL_OUT; 
  out_filter_hook.pf = PF_INET; 
  out_filter_hook.priority = NF_IP_PRI_FIRST; 
  nf_register_hook(&out_filter_hook); 

  in_filter_hook.hook = local_in_hook;
  in_filter_hook.hooknum = NF_INET_LOCAL_IN; 
  in_filter_hook.pf = PF_INET; 
  in_filter_hook.priority = NF_IP_PRI_FIRST; 
  nf_register_hook(&in_filter_hook); 
  return 0;
}

void cleanup_module(void)
{
  nf_unregister_hook(&out_filter_hook); 
  nf_unregister_hook(&in_filter_hook); 
  return 0; 
}
```

We also wrote a Makefile to compile, load and unload our program in the kernal. The target `enable` and `disable` load and unload the module. The target `compile` and `clean` handled the `.ko` files. 

```makefile
TARGET ?= task02
obj-m += ${TARGET}.o

enable: compile
	sudo insmod ${TARGET}.ko

disable:
	sudo rmmod ${TARGET}.ko

list:
	lsmod
	
compile: 	
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:	
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

After loading the module, we used different commands to verify whether the firewall rules in our module worked or not. 

* **RULE01**: The telnet from "<ins style="color:red;font-weight:bold">A</ins>" to "<ins style="color:blue;font-weight:bold">B</ins>" over time. 
* **RULE02**: The output of both `curl -I avirubin.com` and `curl -I 18.216.86.163` were overtime.
* **RULE03**: The output of both `curl -I avirubin.com:443` and `curl -I 18.216.86.163:443` were overtime.4. Block telnet from Machine "<ins style="color:blue;font-weight:bold">B</ins>"
* **RULE04**: The telnet from "<ins style="color:blue;font-weight:bold">B</ins>" to "<ins style="color:red;font-weight:bold">A</ins>" over time. 
* **RULE05**: The ssh from "<ins style="color:blue;font-weight:bold">B</ins>" to "<ins style="color:red;font-weight:bold">A</ins>" over time. 

<figure>
<img style="width:100%; padding-left:0%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_babae38e6d002005a9e634ff21bf5e19.png"/>
<figcaption>Figure 2.1 Verification after setting up the Firewall</figcaption><br>
</figure>

<div style="page-break-after: always;"></div>

## 3. TASK03: Evading Egress Filtering

### 3.1 Preparation before starting

#### 3.1.1 Start up the third Machine

We powered on a new machine "<ins style="color:green;font-weight:bold">C</ins>" in this tasks, and we customised its bash prompt as below. 

<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_401a176ba19cacb3c9897e16ee64e9d0.png"/>
<figcaption>Figure 3.1.1 Configuation for the Machine <ins style="color:green;font-weight:bold">C</ins></figcaption><br>
</figure>

#### 3.1.2 Set up two firewall rules

We used the `ping` and `dig` commands to find the IP address of `www.facebook.com`.

<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_16cdfafe1a05edb561f5e25eb2f94400.png"/>
<figcaption>Figure 3.1.2.1 IP addresses of Facebook in ping and dig</figcaption><br>
</figure>

Next we implemented the below packet filtering program to do the following:
1. Block telnet from Machine "<ins style="color:red;font-weight:bold">A</ins>"
2. Block all outgoing traffic to `www.facebook.com`

```c
static struct nf_hook_ops task03_hook;

unsigned int local_out_hook(
  void * priv, struct sk_buff * skb, 
  const struct nf_hook_state * state
){
  struct iphdr * ip_header = (struct iphdr *)skb_network_header(skb);
  struct tcphdr * tcp_header = (struct tcphdr * )skb_transport_header(skb);
  unsigned int ip_dst_addr = (unsigned int)ip_header->daddr; 

  if (ip_header->protocol != IPPROTO_TCP) return NF_ACCEPT; 
  /* RULE01: Block all the outgoing traffic to external telnet servers */
  if (tcp_header->dest == htons(23)) return NF_DROP;

  /* RULE02: Block all the outgoing traffic to www.facebook.com */
  // 2.1 Block 157.240.229.35
  if (ip_dst_addr == htonl(0x9df0e523)) return NF_DROP;
  // 2.2 Block 31.13.66.35
  if (ip_dst_addr == htonl(0x1f0d4223)) return NF_DROP;
  return NF_ACCEPT; 
}

int init_module(void){ 
  task03_hook.hook = local_out_hook;
  task03_hook.hooknum = NF_INET_LOCAL_OUT; 
  task03_hook.pf = PF_INET; 
  task03_hook.priority = NF_IP_PRI_FIRST; 
  nf_register_hook(&task03_hook); 
  return 0;
}

void cleanup_module(void){
  nf_unregister_hook(&task03_hook);
}
```
We compiled and loaded the module into the kernel. The connections were tested and the results were verified as shown below. 

<figure>
<img style="width:100%; padding-left:0%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_db9448e8d2c421aacb8fa2f33f3fb61e.png"/>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_ef60356e27c07d21f0b9293568f59ac4.png"/>
<figcaption>Figure 3.1.2.2 telnet and www.facebook.com timed-out after configuring <ins style="color:green;font-weight:bold">C</ins></figcaption><br>
</figure>

### 3.2 Task3.a: Telnet to Machine B through the firewall

<!--
* Please describe your observation
* explain how you are able to bypass the egress filtering. 
* You should use Wireshark to see what exactly is happening on the wire.
-->

To bypass the firewall, we establish an SSH tunnel between Machine "<ins style="color:red;font-weight:bold">A</ins>" and "<ins style="color:blue;font-weight:bold">B</ins>". The following command establishes an SSH tunnel between localhost (port 8000) and Machine "<ins style="color:blue;font-weight:bold">B</ins>" (using the default port 22) and when packets come out of "<ins style="color:blue;font-weight:bold">B</ins>"’s end, it will be forwarded to Machine "<ins style="color:blue;font-weight:bold">B</ins>"’s port 23:

```shell
$ ssh -fN -L 8000:10.0.2.7:23 seed@10.0.2.6
```
We now try to establish a telnet connection to Machine "<ins style="color:green;font-weight:bold">C</ins>". 

On Machine "<ins style="color:red;font-weight:bold">A</ins>", the tunnel receives TCP packets from the telnet client (when we do telnet localhost 8000). On receiving this packet, the tunnel forwards the TCP packets to Machine "<ins style="color:blue;font-weight:bold">B</ins>" – port 22. Here, the received data is put into another TCP packet and sent to Machine "<ins style="color:blue;font-weight:bold">B</ins>"’s port 23 (as mentioned in the SSH connection). The firewall only sees the SSH traffic and not the telnet traffic, and hence this SSH tunnel can be used to evade the firewall rule of blocking telnet connections.

<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_081141665c0c33ae63178125aca64d88.png"/>
<figcaption>Figure 3.2.1 Telnet worked well after tunnelling </figcaption><br>
</figure>

We used wireshark to verify the results of this task and observed that no telent packet is being transffered directly from Machine "<ins style="color:red;font-weight:bold">A</ins>" to Machine "<ins style="color:green;font-weight:bold">C</ins>"
<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_955d1cf52ae70d854f763c4585df980e.png"/>
<figcaption>Figure 3.2.2 Packages after establishing the telnet </figcaption>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_25e045983419ddc4aa5b595157fa9a1d.png"/>
<figcaption>Figure 3.2.3 No packages directly transferred from A to C </figcaption><br>
</figure>

### 3.3 Task3.b: Connect to Facebook using SSH Tunnel

<!--
* Please explain what you have observed, especially on why the SSH tunnel can help bypass the egress filtering. 
* You should use Wireshark to see what exactly is happening on the wire. 
* Please describe your observations and explain them using the packets that you have captured.
-->

Another way of evading firewall rule is using Dynamic Port forwarding instead of the static one used in the previous task. In this, we only specify the local port number and not the destination. On the receiver’s end, the packet is forwarded dynamically based on the destination information in the packet. We achieve this using the following command. This command establishes an SSH tunnel between localhost port 9000 and Machine "<ins style="color:blue;font-weight:bold">B</ins>" port 22:

```shell
$ ssh -fN -D 9000 -C seed@10.0.2.6
```
Now, in order to evade the firewall, we need the browser to connect to localhost:9000 whenever it needs to connect to the web server. This will ensure that the traffic goes through the SSH tunnel. To do that, we use localhost:9000 as Firefox’s proxy and set the same as follows:

<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_24bcefb270c85c165d21256cca6060d6.png"/>
<figcaption>Figure 3.3.1 Configuration for Firefox Proxy </figcaption><br>
</figure>

We use the SOCKS proxy because we need to let the browser tell the proxy about the destination in order to dynamically forward the traffic, since this proxy does not have the capability of detecting the destination on its own. After this is set, we visit the blocked web page and see that the website loads without any issues:

<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_dc5673a6e05698dd944731b6ec4832d9.png"/>
<figcaption>Figure 3.3.2 Facebook worked well after configuring  </figcaption><br>

Now, on breaking the SSH tunnel,clearing firefox cache and reloading the page, we see that we are no more able to load the webpage. We see the error that the proxy server is refusing connections. This is because Machine "<ins style="color:red;font-weight:bold">A</ins>" and "<ins style="color:blue;font-weight:bold">B</ins>" are no more connected via SSH, and this SSH tunnel was the proxy server for the browser.

<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_d2d201595bdb8bcad25b27107f621c7a.png"/>
<figcaption>Figure 3.3.3 Connection to Facebook did not work after stop the ssh </figcaption><br>

On establishing the SSH tunnel again and reloading the webpage, we see the web page again as before. Since we have the established SSH tunnel, that is acting as the proxy server, the website was loaded again.

<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_9e065cc16058efe7308a9ad831bc69df.png"/>
<figcaption>Figure 3.3.4 Facebook worked again after restoring  </figcaption><br>
</figure>

On looking at the Wireshark traffic, we see that the traffic to the website goes through Machine "<ins style="color:blue;font-weight:bold">B</ins>"  and not Machine "<ins style="color:red;font-weight:bold">A</ins>" . This is because the web traffic goes via the SSH tunnel between Machine "<ins style="color:red;font-weight:bold">A</ins>"  and Machine "<ins style="color:blue;font-weight:bold">B</ins>" , and Machine "<ins style="color:blue;font-weight:bold">B</ins>"  then forwards the traffic to the destination (web server). Since the traffic goes via SSH and not HTTP anymore, the firewall rule of blocked website is evaded.


<figure>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_65008707297787cde078d49e2557e4de.png"/>
<figcaption>Figure 3.3.5 Packages captured in Wireshark </figcaption><br>
<img style="width:75%; padding-left:12.5%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_c82bcd7229be4ccbc775a118a593b38d.png"/>
<figcaption>Figure 3.3.6 No HTTP package directly out from A  </figcaption><br>
</figure>

## 4. TASK04: Evading Ingress Filtering

<!--
* The objective of this task is to be able to access the web server on Machine A from outside. 
* We will use two machines to emulate the setup. 
    - Machine A is the internal computer, running the protected web server; 
    - Machine B is the outside machine at home. 
* On Machine A, we block Machine B from accessing its port 80 (web server) and 22 (SSH server). 

* You need to set up a reverse SSH tunnel on Machine A, so once you get home, you can still access the protected web server on A from home.
-->
### 4.1 Preparation 

In this task, we used two machines: "<ins style="color:red;font-weight:bold">A</ins>" and "<ins style="color:blue;font-weight:bold">B</ins>". We used the codes below to set up the firewall expected on "<ins style="color:red;font-weight:bold">A</ins>". We implemented two packets filtering rules which block "<ins style="color:blue;font-weight:bold">B</ins>" from accessing its port 80 (web server) and 22 (SSH server). 

```c
static struct nf_hook_ops task04_hook;

unsigned int local_in_hook(
  void * priv, struct sk_buff * skb, 
  const struct nf_hook_state * state
){
  struct iphdr * ip_header = (struct iphdr *)skb_network_header(skb);
  struct tcphdr * tcp_header = (struct tcphdr * )skb_transport_header(skb);
  unsigned int ip_dst_addr = (unsigned int)ip_header->daddr; 

  if (ip_header->protocol != IPPROTO_TCP) return NF_ACCEPT; 
  /* RULE01: Block SSH package from Machine B */
  if (tcp_header->dest == htons(22) // PORT NUMBER 22(SSH)
    && ip_dst_addr == htonl(0x0a000206) // IP ADDR 10.0.2.6
  ) return NF_DROP;

  /* RULE02: Block HTTP package from Machine B */
  if (tcp_header->dest == htons(80) // PORT NUMBER 80(HTTP)
    && ip_dst_addr == htonl(0x0a000206) // IP ADDR 10.0.2.6
  ) return NF_DROP;
  return NF_ACCEPT; 
}

int init_module(void){ 
  task04_hook.hook = local_in_hook;
  task04_hook.hooknum = NF_INET_LOCAL_IN; 
  task04_hook.pf = PF_INET; 
  task04_hook.priority = NF_IP_PRI_FIRST; 
  nf_register_hook(&task04_hook); 
  return 0;
}

void cleanup_module(void){
  nf_unregister_hook(&task04_hook);
}
```

We executed the target `make enable` in previous Makefile to activate the filtering polices. After that, we used the commands`curl -I 10.0.2.15` and `ssh 10.0.2.15` on Machine "<ins style="color:blue;font-weight:bold">B</ins>". The outputs below prove that our codes actually block those packages. 

<figure>
<img style="width:100%; padding-left:0%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_1db9f8d1c8da0a716c0e9ad3a31efee0.png"/>
<figcaption>Figure 4.1.1 Configuration for Firefox Proxy </figcaption><br>
</figure>



### 4.2 Reverse SSH  

Since the Machine blocks only incoming SSH tunnel, we set up a reverse SSH tunnel on Machine "<ins style="color:red;font-weight:bold">A</ins>" which is not blocked by the firewall. We used the command below, which created a reverse SSH tunnel. Specifically, if someone sends a http requests to `10.0.2.6:8000` (Machine "<ins style="color:blue;font-weight:bold">B</ins>"), the request will forward to the internal server on `10.0.2.15:80` (Machine "<ins style="color:red;font-weight:bold">A</ins>"). The responses from the webserver will also be forwarded to the ssh client on the blocked machine by the ssh tunnel. 

```shell
$ ssh -fN -R 8000:10.0.2.15:80 seed@10.0.2.6
```


The images below shows that the previous blocked website was avaible on the `10.0.2.6:8000`(Machine "<ins style="color:blue;font-weight:bold">B</ins>"). We also used the wireshark to capture all packages on `10.0.2.15` (machine "<ins style="color:red;font-weight:bold">A</ins>"). We found that there is only ssh package between the two machines and no http packages directly. Besides, the ssh package to Machine "<ins style="color:blue;font-weight:bold">B</ins>" was not to port 22, which bypass the package filter on the ssh. 

<figure>
<img style="width:100%; padding-left:0%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_95f7b748edcdfcfdee75bce4127c2855.png"/>
<figcaption>Figure 4.1.2 The website was accessable on localhost:8000 after SSH Reversing </figcaption><br>

<img style="width:100%; padding-left:0%; " src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_96b6cdb5f2dc6b83f2e97915466bf75a.png"/>
<figcaption>Figure 4.1.3 The captured packages on (Machine "<ins style="color:blue;font-weight:bold">B</ins>") </figcaption><br>
</figure>
