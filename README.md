# FreePBX LDAP for Gigaset
A simple LDAP server to serve two seperate searchable address books of internal extensions and the contacts from the contact manager to Gigaset DECT devices such as the current N670IP Pro.

## Thanks!
This work is based on a1commss freepbx-ldap, vjeantet's goldap as well as vjeantet's ldapserver. Many THANKS for sharing your work.


## How it works
It starts the LDAP service on port 10389 and on port 11389 and responds to directory search requests by translating them into a SQL query against the "asterisk.users" table in MySQL/MariaDB.

Since we aren't working with sensitive information or trying to implement authentication, but most phones require a bind request with a username & password before they'll search, it'll respond as success to any bind request without checking credentials.

This means the address list will always be up-to-date, as there is no import/export.

Two fields are returned for each result, "displayName" and "telephoneNumber".

MySQL to LDAP mapping is:
* "name" in MySQL maps to "displayName" in LDAP
* "extension" in MySQL maps to "telephoneNumber" in LDAP

## Build & Usage
You can build the binary on your linux dev machine (as long as its the same processor architecture) or directly on your freepbx server (recommended). To build everything on the freepbx server you need to login via ssh an follow the instructions below:

To build, you will need to install the Go runtime (you might need to change the version number if there is a newer Go version)

```
yum update
wget https://dl.google.com/go/go1.15.6.linux-amd64.tar.gz
tar -xzf go1.15.6.linux-amd64.tar.gz
mv go /usr/local
```

Setup the environment
```
export GOROOT=/usr/local/go
export GOPATH=/usr/local/go/bin
```

Clone the repo
```
git clone https://github.com/kaeferfreund/freepbx-ldap.git
```

Clone dependencies
```
go get github.com/vjeantet/ldapserver
go get github.com/lor00x/goldap/message
go get github.com/go-sql-driver/mysql
```

Execute build in the directory of main.go from the cloned freepbx-ldap repo
```
go build
```

Start with debug console output
```
./freepbx-ldap
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
Server port: 10389 <for internal phonebook> and 11389 <for external phonebook>
Name filter: (&(telephoneNumber=*)(displayName=%))
Number filter: (&(telephoneNumber=%)(displayName=*))
Display format: %displayName
Surname: displayName
Phone (office): telephoneNumber
```

# Licensing
Please contact me before using my work in a commercial application
