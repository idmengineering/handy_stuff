# Releasing NameID with a Specified Format #

A frequent task for ADFS Identity Provider administration is onboarding a new Relying Party Trust and releasing to that relying party a particular set of attributes. Frequently, service providers will request a particular attribute take the form of a Name Identifier (NameID), formatted accordingly.

Name Identifiers are special attributes that come within the SAML `<subject>` element. For example,

~~~~
<Subject>
    <NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
      user.name@idmengineering.com
    </NameID>
    <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="_31dde5dfce9c812303ed02d73fb382e9"
                                 NotOnOrAfter="2019-11-06T18:41:11.977Z"
                                 Recipient="https://sp.example.com/SAML2/POST" />
    </SubjectConfirmation>
</Subject>
~~~~

Here, I've released an email address for my user with the format:
`urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`. Let's re-create the above SAML response `<subject>`.

## Example: E-mail Address as NameID ##


For the remainder of this discussion, we'll assume that you've configured a claims-aware relying-party trust within ADFS. We now want to configure a NameID to be released from a particular LDAP attribute. Here's how we'll achieve the above result with ADFS.

*Note: These instructions should work for ADFS 2.0 and up.*

1. From the ADFS Management Console, select **Trust Relationships** > **Relying Party Trusts**.

2. Highlight the relying party which you are trying to configure, and under **Actions** on the right hand side pane, select **Edit Claim Rules**.

3. Click **Add Rule**.

4. The default claim rule template is **Send LDAP Attributes as Claims** so you should select **Next**.

5. Give the rule a meaningful name. We'll first be establishing a claim that we can later use to release the attribute with a particular format, so I've chosen to call this rule "*Make Email Address Available for NameID*".

<p align="center"><img src="http://idmengineering.com/wp-content/uploads/2019/11/adfs-add-claim-for-name-id.png"></p>

6. Select your attribute store, which will most likely be **Active Directory**.

7. Select the attribute that you wish to release as the NameID. Here I will select **Email Addresses**.

8. And select the outgoing claim type as **E-mail Address**.

  - **Note:** Do *NOT* select *Name ID* as the outgoing claim type here if you wish to *specify the format*. Selecting this will send the user's name address as the Name ID, however, there will be no formatting information, i.e. `<NameID>user.name@idmengineering.com</NameID>` would appear within the ``<subject>`` element of the SAML Response.


9. Click **Finish**.

10. Now, we'll add a rule to *Transform* that claim into a properly formatted NameID. Select **Add Rule** once more from the **Edit Claim Rules** dialog.

11. And this time, change the *Claim rule template:* drop-down to **Transform an Incoming Claim**, then select **Next**.

12. Once again, give this rule a meaningful name, like "*Transform Email to NameID*".

13. Select as the **Incoming Claim Type** whatever claim you chose to issue in the previous rule. In this case, that's **E-Mail Address**.

<p align="center"><img src="http://idmengineering.com/wp-content/uploads/2019/11/adfs-transform-claim-for-name-id.png"></p>

14. Select **Name ID** as the outgoing claim type.

15. And now, you can specify the Name ID Format that you wish to use. The following table lists the **Outgoing Name ID Format** selections available within ADFS, and the corresponding format identifier URI that will appear within the SAML `<subject>`.

Outgoing NameID Format | Format Identifier URI
-|-
UPN | http://schemas.xmlsoap.org/claims/UPN
**Email** | **urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress**
Common Name | http://schemas.xmlsoap.org/claims/CommonName
**Unspecified** | **urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified**
X.509 Subject Name | urn:oasis:names:tc:SAML:1.1:nameid-format:X509SubjectName
Windows Qualified Domain Name | urn:oasis:names:tc:SAML:1.1:nameid-format:WindowsDomainQualifiedName
Kerberos Principal Name | urn:oasis:names:tc:SAML:2.0:nameid-format:kerberos
Entity Identifier | urn:oasis:names:tc:SAML:2.0:nameid-format:entity
**Persistent Identifier** | **urn:oasis:names:tc:SAML:2.0:nameid-format:persistent**
**Transient Identifier** | **urn:oasis:names:tc:SAML:2.0:nameid-format:transient**

**Bold** entries represent the most commonly requested formats.

### Cautionary Note About *urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified* ###

The *urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified* format is unique. Formally, this is what the Service Provider should list within their metadata if they **do not care what Name ID Format an IdP uses**. In practice, however, many IdP administrators assume that this specification in an SP's metadata means that they are **REQUIRED** to release a NameID in this format. This is not the case.

It is because of this assumption, however, that ADFS allows one to release a Name ID in this format.

### Persistent vs. Transient *-or-* Welcome to SAML 2.0 ###

For a good discussion of the concepts behind Name Identifiers, see [this post](https://wiki.shibboleth.net/confluence/display/CONCEPT/NameIdentifiers) from the Shibboleth Wiki. In particular, name identifiers have a number of characteristics which define them, including:

- Persistence - whether a given name id is intended to be used across many sessions.
- Revocability - whether a given name id can be revoked.
- Re-assignability - whether a given name id, once revoked, may be reassigned to a different subject.
- Opaqueness - whether a relying party can positively identify the subject from a given name id.
- Targetability - whether a given name id is intended for a specific relying party.
- Portability - whether a given name id is usable across security domains.
- Global - whether a given name id value is globally unique.

The SAML 2.0 spec therefore defines two primary formats:

Type | Characteristics
-|-
Persistent | A persistent, revocable, reassignable, opaque, targeted and portable identifer.
Transient | A non-persistent, non-revocable, opaque, non-portable, non-targeted, global identifier.

which broadly encompass the whole of these Name IDs, and provide a broad basis for (importantly) opaque identifiers. Both NameIDs result in *hashed* values being sent, but one will always be different for every user and every session (transient), and the other will always be consistent for a given user and service. But importantly, neither will be easily associated to a given user based only on the Name ID value.
