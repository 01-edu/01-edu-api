# 01-API

## Packages

### Read-only-access API

- Install

```console
$ npm i @01-edu/api
```

### Instructions

This package allows users to create a read only access to the schools database (keep in mind that this access will only be provided for admins).

<details>
<summary><b>How to get an access token from gitea?</b></summary>

To get an access token from gitea, you must go to **user/settings/application** then **Generate New Token** (https://DOMAIN/user/settings/applications).

1. Go user settings on gitea.

![instructions](./img/setting.png 'Instructions')

2. Then in application you can create the token by inserting a name for this token then it will be generated.

![instructions](./img/access_token.png 'Instructions')

Or you can use `curl` and send a request to gitea:

```sh
$ curl -X 'POST' 'https://someone:123456@git.dev.01-edu.org/api/v1/users/someone/tokens' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"name": "access_token"}'
```

output

```sh
{"id":4,"name":"access_token","sha1":"592cfb612d027eeb45359d837a93b4e22b5e1","token_last_eight":"e22bb5e1"}
```

</details>

<details>
<summary><b>What can you query?</b></summary>
Querying information is depended on the users role. You can see all possible tables that this role can query <a>https://public.01-edu.org/docs/db/db-authorization</a>
</details>

<details>
<summary><b>How to query the information? Where to find examples?</b></summary>
You can take a look into the documentation <a>https://public.01-edu.org/docs/db/graphql</a>.
</details>

---

### **Usage**

This package contains the following exported functions:

- `createClient`
- `singOut`
- `getToken`
- `decode`
- `exportAsCsv`

### `createClient`

This function allows the application to init a client. This client will :

- generate a new **read only token** saving it in cache (global state)
- initialize a refresh loop, where it will refresh the **read only token** whenever the token expires.

Returns the `cache` and a function `run`.\
The `run` function allows the application to query the database. The `cache` contains a set of functions that allows the application to access the local storage (where the token is saved)

example:

```js
const domain = 'dev.01-edu.org'
// access_token is the token provided by gitea
const access_token = '427faa391a0d73a68b69d4d3b65796fd798e9156'

const client = await createClient({
  domain,
  access_token,
})

client.run('query {user{id, login}}').then(console.log)
console.log(client.cache.get('hasura-jwt-token'))
```

output:

```console
{
  user: [
    { id: 1, login: '01-edu' },
    { id: 6, login: 'Joao' }
    { id: 7, login: 'Someone' },
    { id: 8, login: 'Lee' },
  ]
}
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI2IiwiaWF0IjoxN1MzY2LCJpcCI6IjE3Mi4xOC4wLjEsIDE3Mi4xOC4wLjMiLCJleHAiOjE2MjkpbXMiOnsieC1oYXN1cmEtYWxsb3dlZC1yb2xlcyI6WyJ1c2VyIiwiYWRtaW5fcmVhZF9vbmx5Il0sI1kWx0LXJvbGUiOiJhZG1pbl9yZWFkX29ubHkiLCJ4LWhhc3VyYS11c2VyLWlkIjoiNiIsIngtaGFzdXJhLXRva2VuLWlkIjoiZjgzZmM2YTItZWFhNC00NDVmLTgyNmYtYTg1NTgzZjA1NWY3In19.HObIGivW31TOqFNlzu6VY7ACuTC5x0numm6-hOKp0
```

---

### `signOut`

This function allows the application to expire the current **read only token**, and ends the refresh loop

example:

```js
const domain = 'dev.01-edu.org'
const access_token = '427faa391a0d73a68b69d4d3b65796fd798e9156'

const client = await createClient({
  domain,
  access_token,
})

client.run('query {user{id, login}}').then(console.log)

console.log('before signOut: ', client.cache.get('hasura-jwt-token'))
signOut(domain)
console.log('after signOut: ', client.cache.get('hasura-jwt-token'))
console.log('-----')
console.log(await client.run('query {user{id, login}}')))

```

output:

```js
{
  user: [
    { id: 1, login: '01-edu' },
    { id: 6, login: 'Joao' }
    { id: 7, login: 'Someone' },
    { id: 8, login: 'Lee' },
  ]
}
before signOut: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI2IiwiaWF0IjoxN1MzY2LCJpcCI6IjE3Mi4xOC4wLjEsIDE3Mi4xOC4wLjMiLCJleHAiOjE2MjkpbXMiOnsieC1oYXN1cmEtYWxsb3dlZC1yb2xlcyI6WyJ1c2VyIiwiYWRtaW5fcmVhZF9vbmx5Il0sI1kWx0LXJvbGUiOiJhZG1pbl9yZWFkX29ubHkiLCJ4LWhhc3VyYS11c2VyLWlkIjoiNiIsIngtaGFzdXJhLXRva2VuLWlkIjoiZjgzZmM2YTItZWFhNC00NDVmLTgyNmYtYTg1NTgzZjA1NWY3In19.HObIGivW31TOqFNlzu6VY7ACuTC5x0numm6-hOKp0
before signOut: undefined
-----
  {
    message: 'Could not verify JWT: JWSError (CompactDecodeError Invalid number of parts: Expected 3 parts; got 1)'
  }
```

---

### `getToken`

This allows application to generate a new token without the client being initialized.

example:

```js
getToken({ domain, access_token }).then(console.log)
```

output:

```js
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI2IiwiaWF0IjoxNjI4Nzg4OTk1LCJpcCI6IjE3Mi4xOC4wLjEsIDE3Mi4xOC4wLjMiLCJleHAiOjE2MjkyMjA5OTUsImh0dOi8vaGFzdXJhLmlvL2p3dC9jbGFpbXMiOnsieC1oYXN1cmEtYWxsb3dlZC1yb2xlcyI6WyJ1c2VyIiwiYyZWFkX29ubHkiLCJ4LWhhc3VyYS11c2VyLWlkIjoiNiIsIngtaGFzdXJhLXRva2VuLWlkIjoiNTM5NzBkZGItMmIzNi00OTFiLTgwMDMtOTRhZTExMDU0N2U3In19.p6HtlfClZUbLwgbwx8JJs_eSPzGOMEvC0uDDsXtA
{
  sub: '6',
  iat: 1628788995,
  ip: '172.18.0.1, 172.18.0.3',
  exp: 1629220995,
  'https://hasura.io/jwt/claims': {
    'x-hasura-allowed-roles': [ 'user', 'admin_read_only' ],
    'x-hasura-campuses': '{}',
    'x-hasura-default-role': 'admin_read_only',
    'x-hasura-user-id': '6',
    'x-hasura-token-id': '53970dd-2b36-491b-800-94ae1105477'
  }
}
```

---

### `exportAsCsv`

This allows the application to convert the result form the query to a csv format.

example:

```js
const domain = 'dev.01-edu.org'
const access_token = '427faa391a0d73a68b69d4d3b65796fd798e9156'

const client = await createClient({
  domain,
  access_token,
})

client.run('query {user{id, login}}').then((data)=> console.log(exportAsCsv(data)))
```

output:

```console
id,login
1,01-edu
6,Joao
7,Someone
8,Lee
```
