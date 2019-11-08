# LDAP SSL/TLS Config for Shibboleth IdP

Many (if not most) of our clients utilize LDAP as the authentication source for Shibboleth IdP.

That said, we all too often encounter environments without properly configured encryption of the connection. All-to-often this arises because of uncertainty about the options and how to properly configure SSL/TLS for the IdP.

Shibboleth supports two mechanisms for encrypted connections:

## TLS

The most basic connection is TLS (transport layer security). Confusingly, this is usually referred to in logs and configs as SSL (even outside of Shibboleth), but does not refer to true SSL by itself. As a protocol, SSL has [long been deprecated](https://www.globalsign.com/en/blog/ssl-vs-tls-difference/), but we still use the term colloquially to refer to TLS.

The Shibboleth configuration options (in the `ldap.properties` file) for TLS look something like this:

~~~~
idp.authn.LDAP.ldapURL       = ldaps://ldap.example.org:636
idp.authn.LDAP.useStartTLS   = false
idp.authn.LDAP.useSSL        = true
~~~~

We've set `idp.authn.LDAP.useSSL` to `true` to indicate that our IdP is to connect via TLS (I know, it's silly nomenclature), and our `ldapURL` includes both the protocol specification as `ldaps://` and the port number that's typically used for LDAPS connections (TCP/636).

## Start TLS

The second connection type is called StartTLS. In this mode, the connection starts out as clear text communication over the standard port (TCP/389), during which the IdP indicates to the LDAP server that it should communicate via a secure connection. It's at this point that TLS negotiation occurs, and afterwards communication is secure.

The configuration in Shibboleth looks like this:

~~~~
idp.authn.LDAP.ldapURL       = ldap://ldap.example.org:389
idp.authn.LDAP.useStartTLS   = true
idp.authn.LDAP.useSSL        = false
~~~~

Note that the major differences are that we've explicitly flagged to use StartTLS as opposed to SSL, and the LDAP URL includes the non-secure port number (TCP/389) as well as a regular `ldap://` protocol identifier.

-----

There are advantages to either choice. StartTLS can make networking firewall rules simpler, but not all LDAP deployments support it, and so it's not a universal mechanism.

What's important is that it doesn't matter **which** mechanism you choose, so long as you **do choose one**.

No matter how secure you think the internal network is between the IdP server and the LDAP server, you don't want to trust the data between the two traveling in the clear.
