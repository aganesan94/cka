# Authentication

## Authentication

* Usernames & Passwords
* Username & Tokens
* Certificates
* External Authentication Providers - LDAP
* Service Accounts

* Kubernetes does not have the concept called users.
* Kube-api-server manages using the above provided authentication mechanisms


### Authentication thru file system

* 2 formats to create users

  * CSV File containing either of the following formats
    * Password, Username, userid, groupid
    * Security Token, Username, userid, groupid


* This information is passed in as a part of the kube-api-server's cli 
```
--basic-auth-file=<csv-file>
--token-auth-file=<csv-file>
```

Authenticating a user with API Server can be done using the following command

```
# If using plaintext authentication
curl -v -k <url> -u "username:password"

# If using Security Authorization Token
curl -v -k <url> --header "Authorization: Bearer <token>"
```

## Conclusions

* This is not a recommended authentication mechanism.
* This is best achieved using the kubeadm set up using a volume mount.
* Best way is to set up role based authorization for users.