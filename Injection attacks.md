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


