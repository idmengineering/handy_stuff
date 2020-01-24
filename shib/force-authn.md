# Forced Authentication with Shibboleth SP

Single sign-on (SSO) allows users to authenticate once against an Identity Provider (IdP), and get "automatically" logged into one or more Service Providers (SPs), without need to re-authenticate for a period of time determined by the length of the IdP's "session" (often taking the form of a cookie stored in the user's browser).

Some services require an [additional layer of security](https://spaces.at.internet2.edu/display/InCFederation/2013/12/08/ForceAuthn+or+Not). They require the user to enter their login and password every time they enter the application. This is typically due to the nature of the application - i.e. higher-risk data like payroll - and hence the SP can send a SAML AuthnRequest with a special "forceAuthn" flag, like so:

~~~~
<samlp:AuthnRequest xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
                    xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
                    ID="_d7d60924682b543f51b9f6eb9810b75e"
                    IssueInstant=""2020-01-01T00:00:00Z"
                    Destination="http://idp.example.com/idp/profile/SAML2/Redirect/SSO"
                    ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
                    AssertionConsumerServiceURL="http://sp.example.com/Shibboleth.sso/SAML2/POST"
                    Version="2.0"
                    ForceAuthn="true">
  <saml:Issuer>http://sp.example.com/shibboleth</saml:Issuer>
  <samlp:NameIDPolicy Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress" AllowCreate="true"/>
  <samlp:NameIDPolicy AllowCreate="1" />
</samlp:AuthnRequest>
~~~~

In order to [configure Shibboleth SP](https://wiki.shibboleth.net/confluence/display/SP3/ForceAuthn) to generate this kind of request, there are two elements in `shibboleth2.xml` that need to be adjusted:

~~~~
<Sessions lifetime="28800" timeout="3600" relayState="ss:mem" maxTimeSinceAuthn="10"
          checkAddress="false" handlerSSL="true" cookieProps="https">
~~~~

First, in the `<Sessions>` element, set the `maxTimeSinceAuthn` parameter to some short interval. This checks that an actual username and password was entered in the last specified number seconds (the example here sets it to 10 seconds).

Next, in the `<SSO>` block:

~~~~
<SSO entityID="http://idp.example.com/idp/shibboleth" forceAuthn="true"
     discoveryProtocol="SAMLDS" discoveryURL="https://ds.example.org/DS/WAYF">
  SAML2 SAML1
</SSO>
~~~~

add the `forceAuthn` flag set with a value of `true`.

## Step Up Authentication

Note, that these settings can be applied in any application declaration, whether it be the default setting, or you utilize an `ApplicationOverride` to establish a second, logical SP. The latter scenario is often utilized when employing "step-up" authentication.

For example, you may have one or more paths which you'd want to protect with an added layer of security. In this case, you can deploy an `ApplicationOverride` that's also integrated with your partner IdP, and protect *that* content with the second application in Shibboleth. In terms of Apache config, it may look something like this:

~~~~
<Location /secure>
  AuthType shibboleth
  ShibRequestSetting requireSession 1
  ShibRequestSetting entityID http://idp.example.com/idp/shibboleth
  require shib-session
</Location>

<Location /secure/supersecure>
  AuthType shibboleth
  ShibRequestSetting requireSession 1
  ShibRequestSetting applicationId MyForceAuthnApp
  ShibRequestSetting entityID http://idp.example.com/idp/shibboleth
  require shib-session
</Location>
~~~~

where you configure an `ApplicationOverride` with the id `MyForceAuthnApp` to have the relevant `<Sessions>` and `<SSO>` blocks above.
