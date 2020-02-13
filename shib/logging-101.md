# Shibboleth Logging 101

If you're installing, configuring, or managing a single sign-on environment, you will inevitably find yourself wanting (or needing) to understand what's going on under the hood. That's where log files come in. In this space, we've collected some useful general information about Shibboleth logging for both:

- [Shibboleth Identity Provider](#shibboleth-idp-logging)
- [Shibboleth Service Provider](#shibboleth-sp-logging)

---

## Shibboleth IdP Logging

Logging on Shibboleth IdP is implemented via an abstract layer ([SLF4J](http://slf4j.org/)) which directs control of logging to the [Logback](http://logback.qos.ch/) facility. Since the project depends upon these logging implementations, Shibboleth is somewhat beholden to configuring via these external methods. Thankfully, they are relatively generic and highly customizable.

Logging is configured in `%{idp.home}/conf/logback.xml`, where `%{idp.home}` is the location where Shibboleth IdP is installed (typically, and by default, `/opt/shibboleth-idp`). Importantly, you don't usually need to adjust this file unless you want to make specific changes to the logging constructs, e.g. changing the format of the logged strings. Most of the major settings you'll need to adjust can be edited from `%{idp.home}/conf/idp.properties`.

Logs are stored within `%{idp.home}/logs`.

#### Logging Options for `idp.properties`

| variable                | default      | function                                                    |
|-------------------------|--------------|-------------------------------------------------------------|
| idp.loghistory          | 180          | Number of days of logs to keep                              |
| idp.process.appender    | IDP_PROCESS  | Appender to use for diagnostic log                          |
| idp.loglevel.idp        | INFO         | Log level for the IdP proper                                |
| idp.loglevel.ldap       | WARN         | Log level for LDAP events                                   |
| idp.loglevel.messages   | INFO         | Set to DEBUG for protocol message tracing                   |
| idp.loglevel.encryption | INFO         | Set to DEBUG to log cleartext versions of encrypted content |
| idp.loglevel.opensaml   | INFO         | Log level for OpenSAML library classes                      |
| idp.loglevel.props      | INFO         | Set to DEBUG to log runtime properties during startup       |
| idp.loglevel.spring     | ERROR        | Log level for Spring Framework (very chatty)                |
| idp.loglevel.container  | ERROR        | Log level for Tomcat/Jetty (very chatty)                    |
| idp.loglevel.xmlsec     | INFO         | Set to DEBUG for low-level XML Signing/Encryption logging   |

#### "Debug" Logging

When we say "turn up logging to DEBUG" we really mean that you should adjust one or more of the above properties in order to see more useful information. There's no fixed set, but in general:

- If you're doing *any kind of debugging* you should set `idp.loglevel.idp = DEBUG`
- If you want to see the actual SAML assertions, you should use a combination such as:

  ```java
  idp.loglevel.idp = DEBUG
  idp.loglevel.messages = DEBUG
  idp.loglevel.opensaml = DEBUG
  idp.loglevel.encryption = DEBUG
  ```

- If you're working on an issue with a data connector or attribute resolver, you might find:

  ```java
  idp.loglevel.idp = DEBUG
  idp.loglevel.ldap = INFO
  ```

  to be all that you really need, however, you can always take `idp.loglevel.ldap` to DEBUG as well (though be aware, it's quite chatty).

> WARNING: There is no reason to keep debug logging turned on in a production environment. This is especially true if you are capturing *raw* or *decrypted* SAML assertions. Don't do it. Tune things back to default by commenting out your changes when you're done! You have been warned.

More information about Shibboleth IdP Logging can be found [on the wiki](https://wiki.shibboleth.net/confluence/display/IDP30/LoggingConfiguration)!

## Shibboleth SP Logging

The Shibboleth SP software writes to two separate diagnostic log files by default, as configured by the `shibd.logger` and `native.logger` logging setup files. The first file governs most of the interesting "SAML" bits, like assertion receipt, decryption, and attribute resolution. These events will be logged into a file named `shibd.log` within the default log directory (unless modified):

- Linux systems: `/var/log/shibboleth`
- Windows systems: `C:\opt\shibboleth-sp\var\log\shibboleth`

`native.logger` controls messages related to RequestMapping, and more often than not isn't needed. However, once caveat is that on Windows systems using IIS the default configuration leads to no creation of a simple `native.log` file. [This can be easily addressed.](shib/iis-native-logger.md)

#### "Debug" Logging

Overall behavior is specified by the `log4j.rootCategory` parm in `shibd.logger`, which is by default:

```
log4j.rootCategory=INFO, shibd_log, warn_log
```

bumping this to `DEBUG` is minimally necessary for most debugging.

#### Debugging assertions

If you are interested in seeing the SAML assertions themselves, set:

```
log4j.category.OpenSAML.MessageDecoder=DEBUG
log4j.category.OpenSAML.MessageEncoder=DEBUG
```

by un-commenting these lines in `shibd.loggger`.

> WARNING: There is no reason to keep debug logging turned on in a production environment. This is especially true if you are capturing *raw* or *decrypted* SAML assertions. Don't do it. Tune things back to default by commenting out your changes when you're done! You have been warned.

More information about Shibboleth SP Logging can be found [on the wiki](https://wiki.shibboleth.net/confluence/display/SP3/Logging)!
