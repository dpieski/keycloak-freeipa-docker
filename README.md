Keycloak and FreeIPA docker image
=================================

This docker image will setup FreeIPA environment on Fedora 20 and add some example users to FreeIPA. 

Image also contains Keycloak server and it will configure it to use FreeIPA LDAP server as Federation provider and it will enable Kerberos/SPNEGO authentication for Keycloak. In order to participate in Kerberos/SPNEGO you still need to do few steps on your laptop - configure Kerberos client and web browser.

In order to have the image running, you need to do these steps:

Building docker image
---------------------
If you want, you can build docker image by yourself. But you can skip this step and use pre-builded image `mposolda/keycloak-freeipa-server` . If you're going to build it by yourself you need to:

**1** Download keycloak-appliance ZIP distribution of version 1.2.0.Beta1 or newer and put this ZIP file to subdirectory `appliance-dist`

**2)**  Install docker on your laptop if you not already have it. Then if you want, you can build docker image by yourself with command:

```
docker build -t keycloak-freeipa-server .
```

So if you followed this and build the image by yourself, replace image name `mposolda/keycloak-freeipa-server` with your name `keycloak-freeipa-server` in later steps. 


Running docker image
--------------------

**1)** Install docker on your laptop if you not already have it. Then run docker image `mposolda/keycloak-freeipa-server`. You need to expose kerberos ports and also HTTP port of Keycloak server. It's also good to expose LDAP server port (so you can connect remotely to LDAP) and Keycloak debugger port (if you want remote Keycloak debugging from your IDE).

More lines are just for easier readability, but whole command should be single line:

```
docker run --name keycloak-freeipa-server-container -ti -h ipa.example.test 
-e PASSWORD=SomePassword123 -p 20088:88 -p 20088:88/udp -p 29080:9080 
-p 20389:389 -p 28787:8787 mposolda/keycloak-freeipa-server
```

**2)** You will need to access keycloak via `ipa.example.test` server as this matches to HTTP service kerberos principal. In linux you can just put this line to `/etc/hosts` file (on your machine, not docker container):

```
127.0.0.1   ipa.example.test
```

**3)** If you want your web browser to participate in Kerberos authentication, you need to configure Kerberos client. You should first install Kerberos client on your machine. This is platform dependent, so consult documentation of your OS on how to do it. On Fedora, Ubuntu or RHEL you can install just package `freeipa-client`, which installs kerberos client, LDAP client and bunch of other stuff. 

Once client is installed, you can configure Kerberos client configuration file (on Linux it's in `/etc/krb5.conf` ) similarly like this:

```
[libdefaults]
  default_realm = EXAMPLE.TEST
  dns_lookup_realm = false
  dns_lookup_kdc = false
  rdns = false
  ticket_lifetime = 24h
  forwardable = yes

[realms]
  EXAMPLE.TEST = {
    kdc = ipa.example.test:20088
  }

[domain_realm]
  .example.test = EXAMPLE.TEST
  example.test = EXAMPLE.TEST  
```

**4)** Finally configure your browser to participate in Kerberos/SPNEGO authentication flow. You need to allow domain `.example.test` to be trusted. Exact steps are again browser dependent. 

For Firefox see for example [http://www.microhowto.info/howto/configure_firefox_to_authenticate_using_spnego_and_kerberos.html](http://www.microhowto.info/howto/configure_firefox_to_authenticate_using_spnego_and_kerberos.html) . URI `.example.test` must be allowed in `network.negotiate-auth.trusted-uris` config option. 

For Chrome, you just need to run the browser with command similar to this (more details in Chrome documentation):

```
/usr/bin/google-chrome-stable --auth-server-whitelist="ipa.example.test"
```

**5)** Test the integration. First ensure that you have Kerberos ticket available for the FreeIPA EXAMPLE.TEST kerberos realm. In many OS you can achieve this by running command from CMD like:

```
kinit hnelson@EXAMPLE.TEST
```

and provide password `Secret123`

You can check with:
```
klist
```

that you have ticket for user `hnelson`

Now you can simply visit this URL from your browser [http://ipa.example.test:29080/auth/realms/freeipa/account/](http://ipa.example.test:29080/auth/realms/freeipa/account/) and you should be logged in automatically as hnelson in Keycloak account management. You can change you firstName/lastName/email or you can also change your password. Password change will be propagated to LDAP and hence Kerberos too, so next time you can obtain kerberos ticket via kinit with the new password.

Other testing users are:
`jduke` with password `Secret123` or `admin` with password `SomePassword123` (as long as you didn't change PASSWORD variable in "docker run" command above) .

To check the configuration of federation provider, go to [http://ipa.example.test:29080/auth/admin](http://ipa.example.test:29080/auth/admin) , login as admin/admin and verify how is Federation provider configured.

**6** You can stop the docker container by running command:
```
exit
```

from the interactive terminal with docker. Then to start it again, you can use:
```
docker start -ai keycloak-freeipa-server-container
```

The startup will be quite fast as FreeIPA and Keycloak are already finished now.






