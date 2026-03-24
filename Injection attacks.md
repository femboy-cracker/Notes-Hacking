### BASIC SQL INJECTION
#### **SQL Injection 1: Input Box Non-String**

When a user logs in, the application performs the following query:

```plain
SELECT uid, name, profileID, salary, passportNr, email, nickName, password FROM usertable WHERE profileID=10 AND password = 'ce5ca67...'
```

When logging in, the user supplies input to the profileID parameter. For this challenge, the parameter accepts an integer, as can be seen here:

```plain
profileID=10
```

Since there is no input sanitization, it is possible to bypass the login by using any True condition such as the one below as the ProfileID

```plain
1 or 1=1-- -
```

Bypass the login and retrieve the flag.

#### **SQL Injection 2: Input Box String**

This challenge uses the same query as in the previous challenge. However, the parameter expects a string instead of an integer, as can be seen here:

```plain
profileID='10'
```

Since it expects a string, we need to modify our payload to bypass the login slightly. The following line will let us in:

```plain
1' or '1'='1'-- -
```

Bypass the login and retrieve the flag.

#### **SQL Injection 3: URL Injection**

This challenge uses a GET request when submitting the login form, as seen here:

[![](https://assets.tryhackme.com/additional/imgur/9RvTzkK.png)](https://assets.tryhackme.com/additional/imgur/9RvTzkK.png)

The login and the client-side validation can then easily be bypassed by going directly to this URL:

`http://10.49.141.158:5000/sesqli3/login?profileID=-1' or 1=1-- -&password=a`

The browser will automatically urlencode this for us. Urlencoding is needed since the protocol does not support all characters in the request. When urlencoded, the URL looks as follows:

`http://10.49.141.158:5000/sesqli3/login?profileID=-1%27%20or%201=1--%20-&password=a`

The %27 becomes the single quote (') character and %20 becomes a blank space.

#### **SQL Injection 4: POST Injection**

When submitting the login form for this challenge, it uses the POST method. It is possible to either remove/disable the JavaScript validating the login form or submit a valid request and intercept it with a tool such as burp or zap and modify it:

[![](https://assets.tryhackme.com/additional/imgur/LRnr2WQ.png)](https://assets.tryhackme.com/additional/imgur/LRnr2WQ.png)


#### SQL Injection Attack on an UPDATE Statement

 If a injection occurs on an UPDATE statement, the damage can be much more severe as it allows one to change records within the database. In the employee management application, there is an edit profile page as depicted in the following figure.

![](https://assets.tryhackme.com/additional/imgur/cBwDJev.png)

This edit page allows the employees to update their information, but they do not have access to all the available fields, and the user can only change their information. If the form is vulnerable to injection, an attacker can bypass the implemented logic and update fields they are not supposed to, or for other users.

We will now enumerate the database via the UPDATE statement on the profile page. We will assume we have no prior knowledge of the database. By looking at the web page's source code, we can identify potential column names by looking at the name attribute. The columns don't necessarily need to be named this, but there is a good chance of it, and column names such as "email" and "password" are not uncommon and can easily be guessed.

![](https://assets.tryhackme.com/additional/imgur/hFMf6BJ.png)  

To confirm that the form is vulnerable and that we have working column names, we can try to inject something similar to the code below into the nickName and email field:

  

asd',nickName='test',email='hacked

When injecting the malicious payload into the nickName field, only the nickName is updated. When injected into the email field, both fields are updated:

![](https://assets.tryhackme.com/additional/imgur/z5xciWd.png)  

The first test confirmed that the application is vulnerable and that we have the correct column names. If we had the wrong column names, then non of the fields would have been updated. Since both fields are updated after injecting the malicious payload, the original statement likely looks something similar to the following code:

UPDATE <table_name> SET nickName='name', email='email' WHERE <condition>

With this knowledge, we can try to identify what database is in use. There are a few ways to do this, but the easiest way is to ask the database to identify itself. The following queries can be used to identify MySQL, MSSQL, Oracle, and SQLite:

# MySQL and MSSQL

',nickName=@@version,email='

# For Oracle

',nickName=(SELECT banner FROM v$version),email='

# For SQLite

',nickName=sqlite_version(),email='

Injecting the line with "sqlite_version()" into the nickName field shows that we are dealing with SQLite and that the version number is 3.27.2:

![](https://assets.tryhackme.com/additional/imgur/zHesMks.png)  

Knowing what database we are dealing with makes it easier to understand how to construct our malicious queries. We can proceed to enumerate the database by extracting all the tables. In the code below, we perform a subquery to fetch all the tables from database and place them into the nickName field. The subquery is enclosed inside parantheses. The [group_concat() (opens in new tab)](https://sqlite.org/lang_aggfunc.html#groupconcat) function is used to dump all the tables simultaneously.  

> "The group_concat() function returns a string which is the concatenation of all non-NULL values of X. If parameter Y is present then it is used as the separator between instances of X. A comma (",") is used as the separator if Y is omitted. The order of the concatenated elements is arbitrary."

',nickName=(SELECT group_concat(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%'),email='

By injecting the code above, we can see that the only table in the database is called "usertable":

![](https://assets.tryhackme.com/additional/imgur/cqhlsgn.png)  

We can then continue by extract all the column names from the usertable:

',nickName=(SELECT  FROM sqlite_master WHERE type!='meta' AND  NOT NULL AND name ='usertable'),email='

And as can be seen below, the usertable contains the columns: , name, profileID, salary, passportNr, email, nickName, and password:

![](https://assets.tryhackme.com/additional/imgur/Gu1VdgI.png)  

By knowing the names of the columns, we can extract the data we want from the database. For example, the query below will extract profileID, name, and passwords from usertable. The subquery is using the [group_concat() (opens in new tab)](https://sqlite.org/lang_aggfunc.html#groupconcat) function to dump all the information simultaneously, and the [|| (opens in new tab)](https://sqlite.org/lang_expr.html#operators) operator is "concatenate" - it joins together the strings of its operands ([sqlite.org (opens in new tab)](https://sqlite.org/lang_expr.html#operators)).

',nickName=(SELECT group_concat(profileID || "," || name || "," || password || ":") from usertable),email='

![](https://assets.tryhackme.com/additional/imgur/jZG8mJG.png)  

After having dumped the data from the database, we can see that the password is hashed. This means that we will need to identify the correct hash type used if we want to update the password for a user. Using a hash identifier such as hash-identifier, we can identify the hash as SHA256:  

![](https://assets.tryhackme.com/additional/imgur/woCJbL5.png)  

There are multiple ways of generating a sha256 hash. For example, we can use [https://gchq.github.io/CyberChef/ (opens in new tab)](https://gchq.github.io/CyberChef/):

![](https://assets.tryhackme.com/additional/imgur/zaV6o4k.png)  

We can then update the password for the Admin user with the following code:

', password='008c70392e3abfbd0fa47bbc2ed96aa99bd49e159727fcba0f2e6abeb3a9d601' WHERE name='Admin'-- -

##### Task

Log in to the " Injection 5: UPDATE Statement" challenge and exploit the vulnerable profile page to find the flag. The credentials that can be used are:

- profileID: `10`
- password: `toor`

The same enumeration demonstrated for finding tables and column names must be done here since the flag is stored inside another table.
#### 

