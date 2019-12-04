Docker dovecot-ldap
===================

A Docker image running Dovecot on Debian stable ("jessie" at the moment) with the following modules:
* LMTP
* LDAP backend (with bind auth)

This image provide mailbox managment services. You can access those mailboxes using `IMAP` or `POP` protocols. Mails might be delivered using `LMTP` protocol.

Every mailbox access required an authenticated user.
The user database is provided by an external LDAP service.

Interfaces
----------

The image exposes several TCP ports. The `IMAP` and `POP` ports used to access to the mailboxes. The `LMTP` port to ship messages to the mailboxes:

* 143: IMAP port
* 993: IMAPs port
* 110: POP port
* 995: POPs port
* 24: LMTP port

Data persistence
----------------

The image exposes three directories:
* /var/mail: Actual mailboxes are stored in this volume. You should implement some backup strategies on those.
* /etc/ssl/localcerts: Service certificate and keys are stored in this volume. Dovecot is expecting the following PEM files: `/etc/ssl/localcerts/imap.cert.pem` and `/etc/ssl/localcerts/imap.key.pem`. If none are provided, the startup script will generate new keys and self-signed certificate.
* /etc/dovecot: If you want to override the default configurations, you can use this volume to make Dovecot use you files.

Usage
-----

The most simple use would be to start the application like so :

    docker run -d 
    -p 143:143
    --link ldap-container:ldap
    -e LDAP_USER_FIELD="uid"
    -e LDAP_BASE="ou=users,dc=yourdomain,dc=com"
    -e LDAP_BIND_DN="cn=dovecot,dc=example,dc=com"
    -e LDAP_BIND_DNPASS="password"
    teid/dovecot-ldap

However, you should use your own certificate and a data-only container to store the mailboxes

    docker run -d
    -p 993:993
    --link ldap-container:ldap
    --volumes-from imap-certs
    --volumes-from imap-data
    -e LDAP_USER_FIELD="uid"
    -e LDAP_BASE="ou=users,dc=yourdomain,dc=com"
    -e LDAP_BIND_DN="cn=dovecot,dc=example,dc=com"
    -e LDAP_BIND_DNPASS="password"
    teid/dovecot-ldap

The following environment variables allows you to override some LDAP configurations:
* LDAP_BASE: The base dn of the LDAP users
* LDAP_BIND_DN (required): The bind dn of the LDAP user
* LDAP_BIND_DNPASS (required): The bind dn password of the LDAP user
* LDAP_USER_FIELD: The field name of your LDAP users used as `username` field
* SSL_KEY_PATH: The SSL private key path
* SSL_CERT_PATH: The SSL certificate path

*Note: If you are using a custom configuration volume with this variables, your configuration files might be altered. You should not use both features.*

The user are authenticated thanks to the user part of the address (`user@domain`). So the `LDAP_USER_FIELD` should contains the usernames without the address extensions. The LDAP auth will use the bind method