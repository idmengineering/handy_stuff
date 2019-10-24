# Configuring Reloadable Services for Shibboleth IdP #

Shibboleth Identity Provider Version 3 introduced the ability to reload individual services within the Shibboleth framework. Prior to IdP v3, if you wanted to onboard a new Service Provider by adding new `<MetadataProvider>` and `<RelyingParty>` elements, you would be required to restart the servlet container. Now, nearly every class of change that you may need to make to the Shib IdP can be done without restarting Jetty.

## Reloadable Services ##

There are two functional methods to achieve this, either a direct request to the administrative handler URL:

~~~~
http(s)://idp.example.org/idp/profile/admin/reload-service?id=[SOME SERVICE]
~~~~

or by utilizing the included sample shell script which will make this call for you:

~~~~
$ /opt/shibboleth-idp/bin/reload-service.sh -id [SOME SERVICE]
~~~~

In each case `[SOME SERVICE]` represents the name of the Shibboleth service that you would like to reload, i.e. one of the service identifiers (`id`) listed [below](##Services).

## Access Control ##

In order to be able to make requests to the reloadable services identified above, you *must* ensure that you have white-listed the IP address of the host making the call. In the case of the `reload-service.sh` script, that includes ensuring that the interface of the IdP server itself has been whitelisted.

To do this, ensure that the IP addresses are listed within `/opt/shibboleth-idp/conf/access-control.xml` as in the following example:

~~~~
<entry key="AccessByIPAddress">
    <bean id="AccessByIPAddress" parent="shibboleth.IPRangeAccessControl"
        p:allowedRanges="#{ {'127.0.0.1/32', '::1/128'} }" />
</entry>
~~~~

## Services ##

The following services can be reloaded:

| Reloadable Service `id`                    | Function                                                                |
|--------------------------------------------|-------------------------------------------------------------------------|
| shibboleth.RelyingPartyResolverService     | RelyingPartyConfiguration resources for a new or migrated installation. |
| shibboleth.MetadataResolverService         | MetadataConfiguration resources.                                        |
| shibboleth.AttributeResolverService        | AttributeResolverConfiguration resources.                               |
| shibboleth.AttributeFilterService          | AttributeFilterConfiguration resources.                                 |
| shibboleth.NameIdentifierGenerationService | NameIDGenerationConfiguration resources.                                |
| shibboleth.ReloadableAccessControlService  | AccessControlConfiguration resources.                                   |
| shibboleth.ReloadableCASServiceRegistry    | Resources containing ServiceRegistry beans to be reloaded.              |

## Example: Onboarding a new SP ##

In the following examples, we will only use `reload-service.sh`, however you can easily adjust the call to the HTTP `GET` as above.

After you've added the SP's `<MetadataProvider>` element to `/opt/shibboleth-idp/conf/metadata-providers.xml` you should first reload that service:

~~~~
[root@idp.example.org shibboleth-idp]# ./bin/reload-service.sh -id shibboleth.MetadataResolverService

Configuration reloaded for 'shibboleth.MetadataResolverService'
~~~~

Then, after you have added an appropriate attribute filter policy for this entity in `attribute-filter.xml`, you should reload *that* service:

~~~~
[root@idp.example.org shibboleth-idp]# ./bin/reload-service.sh -id shibboleth.AttributeFilterService

Configuration reloaded for 'shibboleth.AttributeFilterService'
~~~~

Lastly, presuming you need to configure a `<RelyingPartyOverride>` for this entity to adjust the particulars of the single sign on integration, you should reload that service:

~~~~
[root@idp.example.org shibboleth-idp]# ./bin/reload-service.sh -id shibboleth.RelyingPartyResolverService

Configuration reloaded for 'shibboleth.RelyingPartyResolverService'
~~~~

## Common Issues ##
