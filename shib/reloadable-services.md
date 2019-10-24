# Configuring Reloadable Services for Shibboleth IdP #

Shibboleth Identity Provider Version 3 introduced the ability to reload individual services within the Shibboleth framework. Prior to IdP v3, if you wanted to onboard a new Service Provider by adding new `<MetadataProvider>` and `<RelyingParty>` elements, you would be required to restart the servlet container. Now, nearly every class of change that you may need to make to the Shib IdP can be done without restarting Jetty.

## Reloadable Services ##

There are two functional methods to achieve this, either a direct request to the administrative handler URL:

~~~~
http(s)://idp.example.org/idp/profile/admin/reload-service?id=[SOME SERVICE
~~~~

or by utilizing the included sample shell script which will make this call for you:

~~~~
/opt/shibboleth-idp/$ ./bin/reload-service.sh -id [SOME SERVICE]
~~~~

In each case `[SOME SERVICE]` represents the name of the Shibboleth service that you would like to reload, i.e. one of

## Access Control ##

In order to be able to make requests to the reloadable services identified above, you *must* ensure that you have white-listed the IP address of the host making the call. In the case of the `reload-service.sh` script, that includes ensuring that the interface of the IdP server itself has been whitelisted.
