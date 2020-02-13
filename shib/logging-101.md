# Shibboleth Logging 101

If you're installing, configuring, or managing a single sign-on environment, you will inevitably find yourself wanting (or needing) to understand what's going on under the hood. That's where log files come in. In this space, we've collected some useful general information about Shibboleth logging (for both the IdP and SP).

## Shibboleth IdP Logging

Logging on Shibboleth IdP is implemented via an abstract layer ([SLF4J](http://slf4j.org/)) which directs control of logging to the [Logback](http://logback.qos.ch/) facility. Since the project depends upon these logging implementations, Shibboleth is somewhat beholden to configuring via these external methods. Thankfully, they are relatively generic and highly customizable.

Logging is configured in `%{idp.home}/conf/logback.xml`, where `%{idp.home}` is the location where Shibboleth IdP is installed (typically, and by default, `/opt/shibboleth-idp`). Importantly, you don't usually need to adjust this file unless you want to make specific changes to the logging constructs, e.g. changing the format of the logged strings. Most of the major settings you'll need to adjust can be edited from `%{idp.home}/conf/idp.properties`.

<center>

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

</center>









## Shibboleth SP Logging
