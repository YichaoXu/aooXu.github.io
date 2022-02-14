# SEED LAB: SQL Injection Report

## Task 1  - Print Alice Profile

We followed the instructions to access the SQL management system. We listed all used commands below.

```shell
# Command for accessing the MYSQL Management System
$ mysql -u root -pseedubuntu
# Command for printing all tables 
mysql-> show tables; 
```

The image below shows what we got after executing these commands. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_844bb5c529d7a4147d7529e6e8d319b0.png"/>

<figcaption>Figure 1. Login Mysql, Users Database and credential Table</figcaption><br>
</figure>

We used the following SQL statement to show Alice's profile information. 

```sql
SELECT * FROM credential WHERE name="Alice";
```

The image below shows what we got after executing the statement. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_e8c5a634533c47d12cd3e8be8fe5879d.png"/>
<figcaption>Figure 2. Alice's profile in credential Table</figcaption><br>
</figure>


<div style="page-break-after: always;"></div>

## Task 2 - SQL Injection Attack on SELECT Statement

### Task 2.1 - SQL Injection Attack from webpage

Because the developer inserts user input directly into the SQL statement, we can use the special character to enforce the system to perform unexpected behaviour. We used constructed username in login-page to logged in as admin. 

Input Parameter `admin'; #`, and password can be anything. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_a7300fcb0967bbb265f7a31d6086ac7c.png"/>
<figcaption>Figure 3. SQL Injection attack from webpage</figcaption><br>
</figure>

As the codes section below show, we used `#` to make all chars behand it be seen as a comment. so the system will directly return all information without authentication. 

```sql
SELECT 
    id, name, eid, salary, birth, ssn, phoneNumber, address, email,
    nickname,Password 
FROM credential 
WHERE name= 'admin'; #' and Password='$hashed_pwd'
```

The image below shows that we successfully access the system as *admin*. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_5e761c31d9a864a409dc4445a22fd188.png"/>
<figcaption>Figure 4. Login Successful, Admin Profile Page</figcaption><br>
</figure>

### Task 2.2 - SQL Injection Attack from command line

We found that the example link to `index.php` will always return a 404 page. We used the `ls` command to list all files in the website root directory `/var/www/SQLInjection/`. As the image below shows, there was no `index.php` and we decided to try send request to the `unsafe_home.php` file (because it is previously mentioned in the document). 

<figure>
<img style="width:100%"https://codimd.s3.shivering-isles.com/demo/uploads/upload_e16f8e34568563248e1c124cf610cbdc.png"/>
<figcaption>Figure 5. Files in the website root dir</figcaption><br>
</figure>

Furthermore, we followed the instruction in the document to convert all special characters in parameters (`admin';#` => `admin%27%3B+%23`) . We also used a pair of single quotes to wrap the request URL. We command we used are shown in the codes section below. 

```shell
$ curl 'http://www.seedlabsqlinjection.com/unsafe_home.php?username=admin%27%3B+%23&Password=anything' > task2.2.html
```

The result of the command was stored at "*task2.2.html*". As the image below shows, we opened that file by Firefox, whose content proven that we successfully launch the attack. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_e56db261db873cc3db7a882d7d969161.png"/>
<figcaption>Figure 6. Login Successful</figcaption><br>
</figure>


### Task 2.3 - Append a new SQL statement

In SQL, semicolon (;) is used to separate two SQL statements. We combined the character with `#` in this task to run expect SQL statements. The code sections shows the username we used in the login-page. 

```sql
a'; delete from credential where id=9; #
```

We expected the username will be identified to two statements and executed respectively. Before the semicolon, the string will complete the first statement for login, and the second statement will be `delete from credential where id=9;`. All chars behind the `#` will be seen as comment. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_b60c35a8812fa5d9d1980228e8edb2ad.png"/>
<figcaption>Figure 7. Injecting two SQL statement with a ';</figcaption><br>
</figure>

However, the website displayed following message and both the sql statements do not execute. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_7bc745fe2b0eb56854495cb5bb7569dd.png"/>
<figcaption>Figure 8. Error in executing ';' separated SQL statements</figcaption><br>
</figure>

It is because the function `query()` is used during the executing, instead of `multi_query()`. The function by definition can only execute one sql statement, and statement with `;` will be seen as invalid one in there.  

<div style="page-break-after: always;"></div>

## Task 3 - SQL Injection Attack on UPDATE Statement

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_b307a9638cda3c84c21fd5eb56f42164.png"/>
<figcaption>Figure 9. Editor Profile Page</figcaption><br>
</figure>

### Task 3.1 - Modify your own salary.

We put the `b' salary=999 where name='Alice'; #` into the nickname field, which will change the salary of Alice after submitting. The statement changed the nickname of Alice to "*b*" and most-importantly increased Alice's salary to '*999*'. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_138922c0d560884955bae238cfbda771.png"/>
<figcaption>Figure 10. SQL Injection on salary column</figcaption><br>
</figure>

Image below shows that our attack successfully changed the salary of Alice to '*999*'. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_eb30e591e100d75c33af8dafc5b6a804.png"/>
<figcaption>Figure 11. Updated Alice Salary to 999</figcaption><br>
</figure>


### Task 3.2 - Modify other people’ salary.

We did the similar thing to Boby (put `b', salary=1 where name='Boby'; #` to nickname), but we changed his salary to "*1*" this time. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_9caafc8319d824f94e94a4935033b35d.png"/>
<figcaption>Figure 12. SQL Injection on Boby (boss) Salary</figcaption><br>
</figure>

After that, we queried SQL table to check whether Boby's salary has been change to 1 or not. The image below shows we success. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_b3bea84140e1ae33cf15c1889267b1f7.png"/>
<figcaption>Figure 13. Boby's new salary</figcaption><br>
</figure>

### Task 3.3 - Modify other people’ password.

We used the value `b', Password=sha('blue') where name='boby'; #` to change Boby's password which is pre-handled by `sha` hasing algorithm.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_1291312c1c0b686e8c7bac8001743ea4.png"/>
<figcaption>Figure 14. Updating Boby's password</figcaption><br>
</figure>

After that, we used the new password Trying to login to Boby's profile using `blue` as the password.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_6538cca25e6fdbc0a9e79c712f426777.png"/>
<figcaption>Figure 15. Successful Login Boby's profile</figcaption><br>
</figure>

<div style="page-break-after: always;"></div>

## Task 4: Countermeasure — Prepared Statement

### Task 4.1 - Redoing Task 2 & 3

#### Redo Task 2.1 - SQL Injection Attack from webpage

We applied prepared statement mechanism in the `unsafe_home.php` to divide the process of sending a SQL statement to the database into two steps. Below is the code snippet of the login page using Bind Params.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_5903d10ee5a8febb8640910478f08731.png"/>
<figcaption>Figure 16. Adding Bind Params to login page</figcaption><br>
</figure>

After that, we tried to launch the attack again in login page using that string, `admin';#`.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_a7300fcb0967bbb265f7a31d6086ac7c.png"/>
<figcaption>Figure 18. Login attempt using SQLi</figcaption><br>
</figure>

However, as the image below shows, it responded an error message, which proven that the statement correctly handled the special characters. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_b9a11bf9ae72635ca6bf979564ef6d48.png"/>
<figcaption>Figure 19. Error msg on login</figcaption><br>
</figure>

#### Redo Task 2.2 - SQL Injection Attack from command line

The prepared statements can also prevent a SQLi attack from the `curl` as well. As image below shows, we tried to use `curl` to inject the malicious code. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_5bd5cca8584207e82e1abd56110ac562.png"/>
<figcaption>Figure 20. Curl command for SQLInjection</figcaption><br>
</figure>

However, it responded that the account information did not exist. 


<figure>
<img style="width:100%" src="media/16333809571918/16335702450309.jpg"/>
<figcaption>Figure 21. No output for Curl Command, SQLi Failed</figcaption><br>
</figure>

#### Redo Task 2.3 - Append a new SQL statement

This task output will be the same as for without Bind Parameters. Due to the fact that the error in command execution was due to api `query()` being used instead of

~~~php
$mysqli->multi_query()
~~~

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_7bc745fe2b0eb56854495cb5bb7569dd.png"/>
<figcaption>Figure 22. Error in executing ';' separated SQL statements</figcaption><br>
</figure>

### Task 4.2: Redo Task 3

#### Redo Task 3.1 - Modify other people’ salary.

Applying prepared statements in `backend` code to solve SQLi injection vulnerability.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_aa17dcfdeb53a5380f15107ad6b3ef12.png"/>
<figcaption>Figure 24. Adding Bind Params to Edit Profile page</figcaption><br>
</figure>

The image below shows that Boby's nickname was set to the attack string. It was correctly handled as an argument but a logic part of statement.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_b0cd1a781976de7babdc144ba6a6cf5b.png"/>
<figcaption>Figure 24. Unable to modify the salary due to Bind Params</figcaption><br>
</figure>

---

#### Redo Task 3.2 - Modify other people’ salary.

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_9caafc8319d824f94e94a4935033b35d.png"/>
<figcaption>Figure 25. SQL Injection on Boby's (boss) Salary</figcaption><br>
</figure>

Similar to previous one, which was handled as the argument. The nickname was our attack string. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_b124376615154cbf46ae674f78af05fd.png"/>
<figcaption>Figure 26. Unable to update Boby's salary</figcaption><br>
</figure>

---

#### Redo Task 3.3 - Modify other people’ password.

We tried to changed Boby's password again using that string. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_1291312c1c0b686e8c7bac8001743ea4.png"/>
<figcaption>Figure 26. SQLi to update Boby's password</figcaption><br>
</figure>

Again as we can see Prepared statements sucessfully prevent SQLi attacks which were working before. The password did not changed after attacking. 

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_eb7111fca186e11c5b83ee81d0408e66.png"/>
<figcaption>Figure 27. Boby's old password</figcaption><br>
</figure>

<figure>
<img style="width:100%" src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_41c1ef09e3827a034037f9db3810be02.png"/>
<figcaption>Figure 28. Boby's new password same as old</figcaption><br>
</figure>

