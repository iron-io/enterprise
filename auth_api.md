auth_api
========

Auth API over LevelDB for IronMQ v3


Objects (json)
========

not all fields are required, TODO document required fields

```
<token> {
  "\_id"
  "user\_id"
  "type"
  "name"
  "token"
  "admin"               bool
}
```

```
<project> {
  "id"
  "user\_id"
  "name"
  "type"
  "partner"
  "status"
  "total\_duration"
  "max\_schedules"
  "schedules\_count"
  "task\_count"
  "hourly\_task\_count"
  "hourly\_time"          time.Time
  "flags"                 map[string]bool
  "shared\_with"          []id
}
```

```
<user> {
  "user\_id"
  "email"
  "password"
  "tokens"                []string
  "status"
  "plan\_worker"
  "flags"                 map[string]interface{}
}
```


Endpoints
=========


##### Authentication
HEADER:  ```Authorization``` alternatively (add query string of ```oauth```)

Request Query String:  ```project_id```


```
GET /1/authentication

response: {

}

code: 200 OR 403
```

#### Login

HEADER: application/json ( no token / oauth )

```
POST /1/authentication

request: {
  email: <email>,
  password: <password>
}

response: {
  user object
}
```

All other endpoints require ```Authorization``` HEADER

#### Tokens

```
POST /1/tokens

request: {
  <token>
}

response: {
  <token>
}
```

```
DELETE /1/tokens/{token_id}

response: {
  msg: success/fail
}
```

#### Users

```
POST /1/users
request: {
	email: 	<insert user email>
  password: <user password>
}

response: {
  <user>
}
```

```
GET /1/users/{user_id_or_email}

response: {
  <user>
}
```

```
PATCH /1/users/{user_id_or_email}

request: {
  email: <optional field>
  password: <optional field>
}

response: {
  <user>
}
```

```
DELETE /1/users/{user_id_or_email}

response: {
  msg: "success/fail"
}
```

#### Projects
```
POST /1/projects
request: {
	name:  <insert project name>
}

response: {
  <project>
}
```

```
GET /1/projects/{project_id}

response: {
  <project>
}
```

```
DELETE /1/projects/{project_id}

response: {
  msg: success/fail
}
```

```
PATCH /1/projects/{project_id}/share
PATCH /1/projects/{project_id}/unshare

request: {
  []user_id
}

