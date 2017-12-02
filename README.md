# Keycloak + Other tests

## [What is Keycloak?](http://www.keycloak.org/about.html)

[IdM](https://en.wikipedia.org/wiki/Identity_management) and SSO server solutions

* [Website](https://www.keycloak.org)
* [Official documentation](http://www.keycloak.org/documentation.html)

## Please note

The following information is used only as a basic management tool without customization. There may be a part of the failure.

### Key Concepts

The detailed description is omitted

* Realm
* [Role](https://keycloak.gitbooks.io/server-adminstration-guide/content/topics/roles.html)
* [Client](https://keycloak.gitbooks.io/server-adminstration-guide/content/topics/clients.html): A Keycloak object (service) that can require user authentication. Can be linked with SAML/OpenID Connect
* [Identity Brokering / Identity Providers](https://keycloak.gitbooks.io/server-adminstration-guide/content/topics/identity-broker.html) (IdP): External authentication services such as Google and Facebook
* [User Storage Federation](https://keycloak.gitbooks.io/server-adminstration-guide/content/topics/user-federation.html): External user DB such as LDAP


### User Settings

* Identifier: UUID (immutable)
* Username (aka User ID)
* Password
    - Temporary password: change force on Login if set
* Email
    - Email verification or not
* First / Last name
* Additional Properties
    - Random key-value value
* Group
* Role
* Creation Date
* Account lockout settings
* Temporary lockout setting (by login failure) 
* OTP
* After login to force action
    - OTP Settings
    - Change profile
    - Change Password
    - Email verification
* IdP Integration
    - IDP Side User identifier
    - Token!
* Integration Service
* Login Session


### Access Control

* There is no ability to restrict which services (SAML/OpenID client) can be accessed by user/group.
* Should be controlled with user information received from the service side.
    - Group, Role, Access policy (OpenID only), additional attributes, etc.

### Audit Log

* User Login related "/administrator act is recorded in a structured form with detailed information.
* No doubt, but after each service, the act is left behind.

### Active Directory linkage

* LDAP -> Keycloak synchronization:
    1. Automatically imported at initial login
    2. Perform a full/incremental sync manually or regularly from the console

* Keycloak -> LDAP synchronization:
    - Reflecting on LDAP when creating, modifying, and deleting accounts.
        - Self service!
    - Group deletion is not reflected

* Keycloak also reflected in AD when password changed.

* Authentication method:
    - LDAP binding
    - Kerberos/SPNEGO SSO (unconfirmed)
    - Kerberos (Form ID/PW)

### Google user authentication (and other Open ID Connect)

* If you have an account with the same email, it will automatically be associated with the IdP after the AD password verification. Since Id/password or IdP mode both can log in.
* Create an AD account by specifying the ID, mail, last name, and password for the first login
    - No validation option for form input.
    - You can preset or reject a value in a script
* Multiple different IDP can be integrated
* Authentication flow can be authenticated directly without user/password/IMAP4 selection form
* Script allows only certain Google Apps domain users
    1. Import the HD attribute into the IdP attribute importer.
       Note: You will only be entered if you are creating a new account with IdP login.
    2. Comparing the properties imported from a script to 1

### Multi factor authentication

* [HOTP](https://tools.ietf.org/html/rfc4226) - Counter/Time mode support
    - hash algorithm, various parameters can be set
    - Works well in Google authenticator
* Can be forced to use OTP regardless of authentication method

### Script-based Authentication

* JS can be set to allow/deny authentication rules. See [Java ScriptEngine](https://docs.oracle.com/javase/7/docs/api/javax/script/ScriptEngine.html)
* It is very difficult to use (especially for debugging)
* Example: see [Snippets/stripemaildomainfromusername.js](Snippets/Stripemaildomainfromusername.js).
* Upon rejection, the error message cannot be selected directly, and you must choose from a few constants (http://www.keycloak.org/docs/javadocs/org/keycloak/authentication/AuthenticationFlowError.html).
* You should not deny access to this service. You can customize the flow of the certification process only, and the flow beyond the service is not customizable.

### UI Customization

* offers themes, localization features, etc.
    - except Korean
* The change in management tools is the title of the top of the login page... Warning Banner is not.
* There are terms and conditions, but no edit page.

### 

* Send mail
    - Check your email address
    - Forgot Password
* OpenID Connect Client auhz function
    - the permission setting for the URL path is likely to be possible.
* Clustering
* TODO: Keycloak LDAP bind account Administrator-> change to service account
* [Keycloak Security Proxy](https://keycloak.gitbooks.io/server-installation-and-configuration/content/v/2.2/topics/proxy.html)
    - Reverse proxy
    - The existing application seems to be able to be certified without modification
* Kerberos/SPNEGO SSO


### Issue

* Admin Login Unlocked very soon

* `LOAD_GROUPS_BY_MEMBER_ATTRIBUTE_RECURSIVELY` of Keycloak User Groups Retrieve Strategy does not work properly with Samba 4.3 set as example [Issue] (https://bugzilla.samba.org/show_bug.cgi?id=10493)

* Provisioning Failure
    - Provisioning sometimes fails due to various issues such as network and timing. Try to reprovision the VM using vagrant, or create a new one.
* Problems connecting to VM with SSH key in Vagrant 1.8.5 [vagrant issue #7610](https://github.com/mitchellh/vagrant/pull/7611)
    - in `/opt/vagrant/embedded/gems/gems/vagrant-1.8.5/plugins/guests/linux/cap/public_key.rb`, [patch](https://github.com/mitchellh/vagrant/pull/7611/files#diff-13c7ed80ab881de1369e3b06c66745e8R57)


## Local setting

Vagrant installation:

```
brew cask install vagrant
```

Launch:

```
vagrant up
```

VM Shell Entry

```
vagrant ssh VMNAME
# ex) vagrant ssh keycloak
```

/etc/hosts:

```
192.168.33.224	dc1.example.com
192.168.33.225	keycloak.example.com
192.168.33.226	ci.example.com
```

## Keycloak settings

* Vagrant VM: `keycloak`

* URL: https://keycloak.example.com

### Management realm: master (default)

* Admin console: https://keycloak.example.com/admin/master/console

* Login: admin
* Password: admin

### Week realm: example

Account management page: https://keycloak.example.com/realms/example/account

## Samba 4 AD settings

* Vagrant VM: `dc1`

* Domain: EXAMPLE.COM
* Login: Administrator
* Password: admin


### Test Account List

* `hsw@syscall.io` / `test`
* `test1@example.com` / `test`
* `test2@example.com` / `test`

## Integration example: Jenkins

* URL: https://ci.example.com

* [SAML Plugin](https://wiki.jenkins-ci.org/display/JENKINS/SAML+Plugin)으로 연동.
* Grant permissions for each group with Jenkins' own settings

## Etc

### Export/Import:

From [Server Administration Guide. Ch 16. Export and Import](https://keycloak.gitbooks.io/server-adminstration-guide/content/topics/export-import.html)

```
/srv/keycloak/bin/standalone.sh  -Dkeycloak.migration.action=export -Dkeycloak.migration.realmName=example -Dkeycloak.migration.provider=dir -Dkeycloak.migration.dir=/vagrant/shared/export
```


___

* [Keycloak](http://www.keycloak.org/)
