# Authentication

## Authentication

* Usernames & Passwords
* Username & Tokens
* Certificates
* External Authentication Providers - LDAP
* Service Accounts

* Kubernetes does not have the concept called users.
* Kube-api-server manages using the above provided authentication mechanisms


### Authentication mechanisms

* Static Password File (CSV File)
  * Password (required)
  * Username (required)
  * UserId (required)
  * group (optional)

```shell
--basic-auth-file=<location-of-csv>
```

* Static Token File

```shell
--token-auth-file=<location-of-csv>
```

Note: The static password file and static token file are not recommended, 

* Certificates
* Identity Services

