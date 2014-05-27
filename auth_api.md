auth_api
========

Auth API over LevelDB for IronMQ v3


Endpoints
=========


##### Authentication
HEADER:  ```Authorization``` alternatively (add query string of ```oauth```)

Request Query String:  ```project_id```


```json
GET /1/authentication

response: {

}

code: 200 OR 403
```

#### Login

HEADER: application/json ( no token / oauth )

```json
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

#### Users

```
POST /1/users
request: {
	email: 	<insert user email>
}

response: {

}
```

#### Projects
```
POST /1/projects
request: {
	name:  <insert project name>
}

response: {

}
```

