# Configuring the CAS Management Webapp

This guide documents how to spin-up the CAS Management Webapp as an [Apache Maven](https://maven.apache.org/) overlay. (You can do it with Gradle as well, but I prefer Maven.)

### 1. Pull down the repo.

~~~~
mkdir /opt/cas/
git clone https://github.com/apereo/cas-management-overlay.git
~~~~

You can browser the repository here: [apareo/cas](apareo/cas).

### 2. Edit `etc/cas/config/management.properties`:

-- Add server names for CAS Management Webapp to authenticate, and details of the Management app itself:

~~~
cas.server.name: https://cas.example.edu
cas.server.prefix: https://cas.example.edu/cas
mgmt.serverName=https://idmutil04.kumc.edu
server.context-path=/cas-management
server.port=9443
~~~

-- Add service registry. NOTE: Must be *exactly* what the CAS server uses, i.e. if you use the following configuration for an LDAP service registry in `cas.properties` on the CAS server, make sure to include this same config blob here. The following example demonstrates the [LDAP Service Registry](https://apereo.github.io/cas/6.0.x/services/LDAP-Service-Management.html)

~~~~
# LDAP Service Registry
cas.serviceRegistry.ldap.ldapUrl=[REDACTED]
cas.serviceRegistry.ldap.baseDn=ou=CasTestServiceRegistry,ou=Services,o=IDVAULT
cas.serviceRegistry.ldap.bindDn=cn=casserviceregistry,ou=AuthAccounts,o=IDVAULT
cas.serviceRegistry.ldap.bindCredential=[REDACTED]
cas.serviceRegistry.ldap.serviceDefinitionAttribute=description
cas.serviceRegistry.ldap.idAttribute=uid
cas.serviceRegistry.ldap.objectClass=casRegisteredService
cas.serviceRegistry.ldap.searchFilter=(%s={0})
cas.serviceRegistry.ldap.loadFilter=(objectClass=%s)
~~~~

### 3. Edit the `pom.xml`.

Include any additional dependencies you may have. For example, to include the LDAP Service Registry dependency:

~~~~
<!-- LDAP Service Registry -->
<dependency>
  <groupId>org.apereo.cas</groupId>
  <artifactId>cas-server-support-ldap-service-registry</artifactId>
  <version>${cas.version}</version>
  <exclusions>
    <exclusion>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </exclusion>
  </exclusions>
</dependency>
~~~~

Note that I needed to exclude the dependency's dependency on spring-web so that it wasn't included twice, otherwise you may get an error on startup such as this:

~~~~
More than one fragment with the name [spring_web] was found. This is not legal with relative ordering. See section 8.2.2 2c of the Servlet specification for details. Consider using absolute ordering.
~~~~

See: https://stackoverflow.com/questions/55177367/more-than-one-fragment-with-the-name-spring-web-was-found-in-non-maven-projec

### 4. Edit `etc/cas/config/users.properties` to add your users, i.e.

~~~~
kellen=foo,ROLE_ADMIN
~~~~

Note: The 'foo' is for a legacy password field, can be anything there. This is obviated if you use a JSON or YAML authorization [list](https://apereo.github.io/cas-management/5.3.x/installation/Installing-ServicesMgmt-Webapp.html#authorization).

### 5. Build and deploy the WAR file to Tomcat:

~~~~
./build.sh package
sudo install -C -m 775 -o tomcat -g root etc/cas/config/* /etc/cas/config
sudo install -C -m 775 -o tomcat -g root target/cas-management.war /opt/tomcat/webapps
sudo systemctl start tomcat
~~~~

## Resources:

https://apereo.github.io/cas-management/5.3.x/installation/Installing-ServicesMgmt-Webapp.html
