# Assessing Attribute Release Policies with AACLI

Shibboleth Identity Provider (IdP) includes an incredibly useful and powerful tool for determining, without doing an actual authentication sequence, what attributes will be released for a given user (principal) and service provider.

That tool is the [**A**ttribute **A**uthority **C**ommand **L**ine **I**nterface (AACLI)](https://wiki.shibboleth.net/confluence/display/IDP30/AACLI).

You can invoke the AACLI tool by executing a script in the terminal, i.e. `{idp.home}/bin/aacli.sh` (or `{idp.home}/bin/aacli.bat` for Windows installations):

```
[user @ /opt/shibboleth-idp/]$ .bin/aacli.sh -n user -r https://sp.idmengineering.com/shibboleth
```

If you have correctly configured [Access Controls](https://wiki.shibboleth.net/confluence/display/IDP30/AccessControlConfiguration) for [Administrative Functions](https://wiki.shibboleth.net/confluence/display/IDP30/AdministrativeConfiguration), you may access the output of the script via a special `resolvertest` endpoint as such:

```
https://idp.idmengineering.com/idp/profile/admin/resolvertest?principal=jdoe&requester=https%3A%2F%2Fsp.example.org%2Fsp
``` 

This URL you could access via `curl` for use in custom scripting.

---

## Parameters

| Query Parameter | Shell Flag       | Description                                                                             |
| --------------- | :--------------- | --------------------------------------------------------------------------------------- |
| principal       | --principal, -n  | names the user for which to simulate an attribute resolution.                           |
| requester       | --requester, -r  | identifies the service provider for which to simulate an attribute resolution.          |
| acsIndex        | --acsIndex, -i   | Identifies the index of an `<AttributeConsumingService>` element in the SP's metadata.  |
| saml2           | --saml2          | Value is ignored, if present causes the output to be encoded into a SAML 2.0 assertion. |
| saml1           | --saml1          | Value is ignored, if present causes the output to be encoded into a SAML 1.1 assertion. |

---

## Examples