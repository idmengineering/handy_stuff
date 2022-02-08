# SP Metadata for Amazon Cognito

Cognito is the easy-to-implement authentication service for web and mobile apps hosted in the AWS ecosystem. 

Cognito provides "user pools" &mdash; or groups of user's coming from various sources &mdash; against which an application can authenticate a user, with those further able to be extended to external sources such as social media (Google, Facebook, Amazon) or federated identity providers via SAML 2.0.

And when it comes to implementing SAML 2.0 integration with an identity provider (IDP), Amazon provides pretty good [documentation](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-managing-saml-idp-console.html). 

However that documentation, and indeed the Cognito service, lacks something relatively fundamental.

Cognito admirably accounts for the fact that most Service Provider operators will recieve from their integration partners an XML metadata bundle representing the IDP, and hence provides the ability to configure the SAML connection on the Cognito side by uploading that IDP metadata document. They even allow you to supply a link to the IDP metadata, given that many IDP operators maintain URLs which serve up a signed copy of the latest metadata, in an effort to provide simpler rollover of SAML signing and encryption certificates.

However, where Cognito fails utterly (as of the writing of this document) is to provide a simple means to generate service provider (SP) metadata, leading to awkward conversations where unknowing Cognito admins are being asked by more knowledgeable IDP operators for metadata that they don't have. Furthermore, Cognito's documentation is really lacking in the area of **how** to create that metadata.

As such, this post is intended as a quick how-to for Cognito SP operators to generate valid XML metadata representing the Cognito SP.

## Metadata Prerequisites

There are three core pieces of information that you are **required** to know in order to generate SAML SP metadata for a Cognito User Pool:

#### The Cognito User Pool ID: `$pool_id`

You can access this information from the AWS Console. From the upper left hand corner select "Services -> Security, Identity, and Governance -> Cognito" to access the Cognito control panel. Then select "Manage User Pools". You should then select the User Pool for which you wish to obtain the Pool ID. Selecting a User Pool will take you to the "General Settings" for the Pool, which should list the "Pool ID" at the top:

<p align="center">
    <img src="https://idmengineering.com/resources/docimages/vS8GDeW.png">
</p>

At the time of writing this post Cognito is moving to a "new" interface. You can access the Pool ID in basically the same way, however Amazon conveniently lists the Pool ID on your list of User Pools:

<p align="center">
    <img src="https://idmengineering.com/resources/docimages/2NzzjlM.png">
</p>

#### The User Pool's AWS Region: `$region`

Conveniently, you can get this straight out of the Pool ID: 

<p align="center">
    <img src="https://i.imgur.com/PSMFHmH.png">
</p>

as the struction of the `$pool_id` is `$region_XXXXXXXXX`. As you can see from the above example, my test User Pool is located in the "US East 2 (Ohio)" region, hence `$region = us-east-2` for our metadata.

#### Cognito Domain Prefix: `$domain_prefix` (or Custom Domain)

Lastly, you will specify a domain prefix when you create the User Pool, which establishes a domain that uniquely identifies the pool to AWS. This will take the form of:

```php
$domain_prefix.auth.$region.amazoncognito.com
```

Alternatively, if you have associated a custom domain to your Cognito User Pool, you will substitute that.

You'll find the settings for the domains under "App Integration -> Domain Name" in the current Cognito User Pool Settings, 

<p align="center">
    <img src="https://i.imgur.com/IfKzzDX.png">
</p>

or on the "App Integration" tab in the new Cognito interface:

<p align="center">
    <img src="https://idmengineering.com/resources/docimages/gxYdKjd.png">
</p>

## Integration Particulars

Additionally, you'll want to make some decisions now about things like the SAML attributes that you require from the IDP. Each of these will be enumated within the templated metadata below, and you'll want to know the `name` of the attribute, as well as it's `friendlyName`.

There are many particularities to Cognito attribute mappings, and fortunately, the [documentation about attribute mapping](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-specifying-attribute-mapping.html) is quite robust. 

The keys are the following:

- Cognito requires an attribute as the SAML <NameID> which will be used to uniquely identify the user within Cognito. You don't necessarily need to use this as the **principal** identifier within your application, but it is ideally a useful identifier for a human being to look at (i.e. not a transient: `urn:oasis:names:tc:SAML:2.0:nameid-format:transient`). We recommend requesting some form of "user id" as a persistent identifier (`urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`).

- Ensure that any attribute mappings that you define in Cognito are properly enumerated within the metadata, as this will assist the IDP deployer in facilitating their configuration of attribute release.

We furthermore strongly encourage the following best practices:

- Don't ask for an email address as the NameID / principal identifiers. Email addresses frequently change!

- Request only the minimal set of attributes your application requires.

- Be flexible! It is generally considered rude within the SAML community for a vendor to demand that an IDP releases a custom attribute specific to your organization. Instead, adapt your attribute mapping to work with what the IDP has available to send. Generic names are also good: `mail`, `uid`, `sn`, `givenName` etc. 

  - Really any of the common [LDAP attributes with their associated OIDs](https://ldap.com/ldap-oid-reference-guide/) are likely to be well-supported by most identity providers.

  - Many IDPs also can easily facilitate common [ADFS attributes](http://docs.oasis-open.org/imi/identity/v1.0/os/identity-1.0-spec-os.html#_Toc229451870).

## Building the SP Metadata
