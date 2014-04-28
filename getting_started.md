# IronMQ On Premise

## Installation

### 1. Unpack the provided archive

You will end up with a directory called `ironmq`, cd into that directory to
continue. 

### 2. Run install script:

```
./iron install
```

### 3. Start Services

This allows you to try it out:

```
./iron start
```

IronMQ will be running at http://localhost:8080 by default.

Keep that running while you do the next steps.

### 4. Create a new user and project

Keep the services running, then **in a new terminal window**, run the following commands
to create a new user and a new project.  

```
./iron -t superToken123 create user somebody@somewhere.com
```

Grab the token that's printed after previous command, you'll need it for the
next ones.

```
./iron -t NEWTOKEN create project myproject
```

Then you can use that new project with the new token to use the API.


### 5. Change configuration

You may want to update the IronMQ port to 80 for instance or change
data directory.

- edit ironauth/config.json and/or ironmq/config.json

Note: be sure super_user.token from ironauth config matches super_token in ironmq config. 

### 6. Ready for production? Setup upstart

```
sudo ./iron upstart
```

To restart with upstart:

```
sudo restart ironmq
sudo restart ironauth
```

## Upgrading

- Get the latest zip package and unzip it
- Run `./iron install`
- Run `sudo restart ironauth`
- Run `sudo restart ironmq`
