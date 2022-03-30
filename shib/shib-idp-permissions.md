# Shibboleth IDP File Permissions

In light of recent developments in the thread landscape involving JNDI, and Java library vulnerabilities such as [**Log4Shell** (CVE-2021-44228)](https://idmengineering.com/log4j_rce_vulnerability_cve-2021-44228/), we note the critical importance of file permissions to the Shibboleth deployment.

> Note: This document currently discusses only [Linux-based](https://shibboleth.atlassian.net/wiki/spaces/IDP4/pages/1265631502/Installation) deployments of Shibboleth IDP. We believe that the [Windows](https://shibboleth.atlassian.net/wiki/spaces/IDP4/pages/1265631507/WindowsInstallation) installation handles permissions specific to that system automatically when performing an MSI-based installation procedure outlined by the wiki. We are investigating best practices for Windows installations and will update this document accordingly with our findings.

## Linux Service Privileges

For a proper, secure installation of Shibboleth IDP, you should configure the servlet container (Jetty) to run as a non-root user. The proper configuration for this depends upon the servlet container of choice, i.e. for Eclipse Jetty (the recommended container):

- [Jetty 9.x - Setting Port 80 Access for a Non-Root User](https://www.eclipse.org/jetty/documentation/jetty-9/index.html#setting-port80-access) 

## Linux Filesystem Permissions

The most critical element of the file system permissions is to ensure that the user which runs the servlet container does *not* have write permissions for the IDP configuration files, except in select circumstances.

Below you'll find a small shell script to set appropriate permissions for a "standard" deployment, meaning:

- all files are by-default owned by `root`,
- select permissions are provided to the user account under which the servlet container runs, i.e. `jetty`,
- Shibboleth is installed in the default location: `/opt/shibboleth-idp`, and
- the `$JETT_BASE` directory of the Jetty installation is `/opt/jetty-base`

```bash
#!/bin/sh
chown -R root:jetty /opt/shibboleth-idp/;
chown -R jetty:jetty /opt/shibboleth-idp/{credentials,logs,metadata};
find /opt/shibboleth-idp -type d -exec chmod 750 {} \;
find /opt/shibboleth-idp -type f -exec chmod 640 {} \;
chmod -R 750 /opt/shibboleth-idp/bin;
chmod -R u-w /opt/shibboleth-idp/credentials;
chown -R root:jetty /opt/jetty-base/;
chown -R jetty:jetty /opt/jetty-base/{logs,tmp};
find /opt/jetty-base -type d -exec chmod 750 {} \;
find /opt/jetty-base -type f -exec chmod 640 {} \;
```

Essentially, the `jetty` user requires read-access to most files within the IDP installation directory, hence we set `root` to own those files and `jetty` as the group, with permissions for the owner (`root`) to be allowed to edit those files, and the group (`jetty`) permission to read. Other users have no access.

`jetty` needs to own the `credentials`, `logs`, and `metadata` directories, with write permissions only for `logs` and `metadata`. If `jetty` doesn't own `credentials` it's unable to unlock the cryptographic keys required for SAML, and will not start. We remove the `jetty` users permissions to write to those files separately (`chmod -R u-w /opt/shibboleth-idp/credentials`).

Likewise, `jetty` needs to be able to write to the `logs` and `tmp` directories from `$JETTY_BASE`.

#### Essentially, the goal is to allow the `jetty` user to have the absolute minimal permissions to run the IDP software. 

>N.B. With this goal in mind, we note that if you do not leverage any `<MetadataProvider>` elements which fetch metadata from external locations, you shouldn't even need to allow write access for the `metadata` directory. One option is to ensure that you specify a `backingFilePath` to a directory other than `<IDP-HOME>/metadata`, i.e. something like `/var/cache/shibboleth`. Then only that directory needs write access for `jetty`.