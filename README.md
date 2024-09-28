# FreePBX LDAP for Gigaset
A simple LDAP server to serve two seperate searchable address books of internal extensions and the contacts from the contact manager to Gigaset DECT devices such as the current N670IP Pro.

## Thanks!
This work is based on a1commss freepbx-ldap, vjeantet's goldap as well as vjeantet's ldapserver. Many THANKS for sharing your work.


## How it works
It starts the LDAP service on port 10389 and responds to directory search requests by translating them into a SQL query against a union of the "asterisk.users", "asterisk.meetme" and "asterisk.ringgroups" tables in MySQL/MariaDB.

Since we aren't working with sensitive information or trying to implement authentication, but most phones require a bind request with a username & password before they'll search, it'll respond as success to any bind request without checking credentials.

This means the address list will always be up-to-date, as there is no import/export.

Two fields are returned for each result, "displayName" and "telephoneNumber".

MySQL to LDAP mapping is:
* "name" in MySQL maps to "displayName" in LDAP
* "extension" in MySQL maps to "telephoneNumber" in LDAP

## Build & Usage
You can build the binary on your linux dev machine (as long as its the same processor architecture) or directly on your freepbx server (recommended). To build everything on the freepbx server you need to login via ssh an follow the instructions below:

To build, you will need to install the Go runtime

```
yum update
wget https://dl.google.com/go/go1.21.4.linux-amd64.tar.gz
tar -xzf go1.21.4.linux-amd64.tar.gz
mv go /usr/local
```

Setup the environment
```
export GOROOT=/usr/local/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

Clone the repo
```
git clone https://github.com/kaeferfreund/freepbx-ldap.git
```

Creat go.mod file in freepbx-ldap folder with these contents
```
module .git/config

go 1.21.4

require (
	github.com/go-sql-driver/mysql v1.7.1
	github.com/lor00x/goldap v0.0.0-20180618054307-a546dffdd1a3
	github.com/vjeantet/ldapserver v1.0.1
)
```

Clone dependencies
```
go get .git/config
```

build
```
go install
go build
mv config freepbx-ldap
```

start with debug console output
```
.\freepbx-ldap
```


## Recommended Install Procedure for production use
```
# mkdir -p /opt/freepbx-ldap
# cp <freepbx-ldap binary location> /opt/freepbx-ldap/freepbx-ldap
# chown -R asterisk:asterisk /opt/freepbx-ldap
# chmod +x /opt/freepbx-ldap/freepbx-ldap
# cp <systemd/freepbx-ldap.service location> /etc/systemd/system/freepbx-ldap.service
# systemctl daemon-reload
# systemctl enable freepbx-ldap
# systemctl start freepbx-ldap
```

## Phone Configuration
You'll need to configure your IP phones to look up against the LDAP server.


### Gigaset N670IP
```
Server port: 10389
Name filter: (&(telephoneNumber=*)(displayName=%))
Number filter: (&(telephoneNumber=%)(displayName=*))
Display format: %displayName
Surname: displayName
Phone (office): telephoneNumber
```

# Licensing
Please contact me before using my work in a commercial application
