# Assessing Attribute Release Policies with AACLI

Shibboleth Identity Provider (IdP) includes an incredibly useful and powerful tool for determining, without doing an actual authentication sequence, what attributes will be released for a given user (principal) and service provider.

That tool is the [**A**ttribute **A**uthority **C**ommand **L**ine **I**nterface (AACLI)](https://wiki.shibboleth.net/confluence/display/IDP30/AACLI).

You can invoke the AACLI tool by executing a script in the terminal, i.e. `{idp.home}/bin/aacli.sh` (or `{idp.home}/bin/aacli.bat` for Windows installations):

```
[user @ /opt/shibboleth-idp/]$ .bin/aacli.sh -n user -r https://sp.idmengineering.com/shibboleth
```

If you have correctly configured [Access Controls](https://wiki.shibboleth.net/confluence/display/IDP30/AccessControlConfiguration) for [Administrative Functions](https://wiki.shibboleth.net/confluence/display/IDP30/AdministrativeConfiguration), you may access the output of the script via a special `resolvertest` endpoint as such:

```
https://shibdev3.idmintegration.com/idp/profile/admin/resolvertest?principal=jdoe&requester=https%3A%2F%2Fsp.example.org%2Fsp
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

### Shell Script, Simple Output

```[root@web2.shibdev.idmintegration.com shibboleth-idp]# ./bin/aacli.sh -n kellen -r https://kellen-shibdev.lab.idmintegration.com/shibboleth```

Result:

```
{
"requester": "https://kellen-shibdev.lab.idmintegration.com/shibboleth",
"principal": "kellen",
"attributes": [


  {
    "name": "uid",
    "values": [
              "kellen"          ]
  },

  {
    "name": "mail",
    "values": [
              "kellen@idmintegration.com"          ]
  },

  {
    "name": "sn",
    "values": [
              "Murphy"          ]
  }

]
}
```

---

### Shell Script, Output formatted as SAML 2.0 `<Assertion>`

```[root@web2.shibdev.idmintegration.com shibboleth-idp]# ./bin/aacli.sh -n kellen -r https://kellen-shibdev.lab.idmintegration.com/shibboleth```

Result:

```
<?xml version="1.0" encoding="UTF-8"?>
<saml2:Assertion ID="_057aa390d3cebb0d9c7b90524667edd1"
    IssueInstant="2020-09-18T16:20:55.242Z" Version="2.0" xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">
    <saml2:Issuer>https://shibdev3.idmintegration.com/idp/shibboleth</saml2:Issuer>
    <saml2:Subject>
        <saml2:NameID
            Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient"
            NameQualifier="https://shibdev3.idmintegration.com/idp/shibboleth" SPNameQualifier="https://kellen-shibdev.lab.idmintegration.com/shibboleth">AAdzZWNyZXQx49glc8r4c80yYO2LWKJ9yHk4GV3IzMIZvBYsEKNnbmxuRfySoLSAZBu7H3OTxNzJKTPIpTJ0o2Ye9YnyMIve0at0+QWNSGz/Rjuu1PW/wvse24m40MFlYWQoWu2EDO5cmYWYUWze/jBPtuyCN0XqM6MJczyAujM=</saml2:NameID>
    </saml2:Subject>
    <saml2:AttributeStatement>
        <saml2:Attribute FriendlyName="uid"
            Name="urn:oid:0.9.2342.19200300.100.1.1" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
            <saml2:AttributeValue>kellen</saml2:AttributeValue>
        </saml2:Attribute>
        <saml2:Attribute FriendlyName="mail"
            Name="urn:oid:0.9.2342.19200300.100.1.3" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
            <saml2:AttributeValue>kellen@idmintegration.com</saml2:AttributeValue>
        </saml2:Attribute>
        <saml2:Attribute FriendlyName="sn" Name="urn:oid:2.5.4.4" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
            <saml2:AttributeValue>Murphy</saml2:AttributeValue>
        </saml2:Attribute>
    </saml2:AttributeStatement>
</saml2:Assertion>
```

---

### URL Request, Simple Output

URL: `https://shibdev3.idmintegration.com/idp/profile/admin/resolvertest?principal=kellen&requester=https%3A%2F%2Fkellen-shibdev.lab.idmintegration.com%2Fshibboleth`

<p align="center"><img src="https://idmengineering.com/images/T8yMbzR.png"></p>

---

### URL Request,  Output formatted as SAML 2.0 `<Assertion>`

URL: `https://shibdev3.idmintegration.com/idp/profile/admin/resolvertest?saml&principal=kellen&requester=https%3A%2F%2Fkellen-shibdev.lab.idmintegration.com%2Fshibboleth`

<p align="center"><img src="https://idmengineering.com/images/BTX779y.png"></p>

---

# Attribute Resolution Trouble?

You can use AACLI to debug issues related to attribute release... but **why** is a given attribute not being released? Here are some common issues:

- The attribute isn't being provided by a data connector. This is perhaps because the attribute is `null` for that principal.
- There is no attribute definition defined for that attribute. 
- The attribute definition does not define a dependency from which to pull the source attribute (i.e. explicitly specify the attribute or say which resolver it's from).
- The attribute definition is marked as a dependency only attribute and thus is not released from the resolver.
- The attribute definition does not define an encoder appropriate for the given request protocol (i.e. SAML1 encoder exists but SAML2 doesn't).
- The attribute is being filtered out by the attribute filter policy. 

The last point, the lack of an appropriate policy releasing the attribute for a given SP in `attribute-filter.xml` is the *most likely* cause of a missing attribute, and as such should most likely be checked first.
