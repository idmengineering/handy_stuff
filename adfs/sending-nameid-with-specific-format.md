# Releasing NameID with a Specified Format in ADFS #

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
`urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`.

## Name Identifiers and ADFS ##

For the remainder of this discussion, we'll assume that you've configured a claims-aware relying-party trust within ADFS. We now want to configure a NameID to be released from a particular LDAP attribute. Here's how we'll achieve the above result with ADFS.

*Note: These instructions should work for ADFS 2.0 and up.*

1. From the ADFS Management Console, select **Trust Relationships** > **Relying Party Trusts**.

2. Highlight the relying party which you are trying to configure, and under **Actions** on the right hand side pane, select **Edit Claim Rules**.

3. Click **Add Rule**.

4. The default claim rule template is **Send LDAP Attributes as Claims** so you should select **Next**.

5. Give the rule a meaningful name. We'll first be establishing a claim that we can later use to release the attribute with a particular format.

<span style="text-align:center;"><img src="http://idmengineering.com/wp-content/uploads/2019/11/adfs-add-claim-for-name-id.png"></span> 
