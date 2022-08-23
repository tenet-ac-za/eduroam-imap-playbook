> *This playbook was developed in support of a [TNC18](https://tnc18.geant.org/) lightning talk [about using GSuite for eduroam](https://tnc18.geant.org/core/presentation/226.html). However, since then, a substantially more scalable alternative has become available: [geteduroam](https://github.com/geteduroam). While the ideas here may be useful (hence archiving the respository), you're strongly encouraged to look to geteduroam for any production use.*

# eduroam-imap-playbook Ansible playbook

This is an [Ansible playbook](http://docs.ansible.com/ansible/latest/playbooks.html) to set up [eduroam](https://eduroam.org/) authentication against an IMAP server.

Our main use-case for this is allowing [Google GSuite](https://gsuite.google.com/) users to authenticate on eduroam. However, the concept should be extensible to any IMAP server.

_**Note:** The original use case for this was for small organisations that only had hosted infrastructure. However that use case has now largely been met by the [eduroam Managed IdP](https://www.eduroam.org/eduroam-managed-idp/) service. You are strongly encouraged to look at Managed IdP rather than deploying this playbook._

## Using the playbook

All configuration happens in [group_vars/all](group_vars/all). You should edit that file to suit your enviroment.

The playbook has been set up to be readily used in a [Vagrant](https://www.vagrantup.com/) enviroment with [VirtualBox](https://www.virtualbox.org/) as the provider. In such an enviroment, the following should get you going:

```bash
git clone https://github.com/safire-ac-za/eduroam-imap-playbook.git
cd eduroam-imap-playbook
vagrant up
vagrant ssh eduroam-imap
```
In other enviroments, the [Vagrantfile](Vagrantfile) may need to be edited. Alternatively, the playbook should be usable without Vagrant given an appropriate [inventory](inventories/).

The playbook is also (hopefully) self documenting, allowing people who don't make use of Ansible to follow the configuration.

## Roles

This playbook contains a number of roles, all of which should be usable independently and outside of the playbook.

### [pam-imap](roles/pam-imap)

The **pam-imap** role downloads, compiles, installs, and configures the [pam-imap](https://github.com/wdoekes/pam-imap) pluggable authentication module. This PAM module allows for authentication against an IMAP server. By default, this is configured to be `imap.gmail.com`. See the [documentation](roles/pam-imap/README.md) for how to change this.

### [freeradius-eduroam](roles/freeradius-eduroam)

The **freeradius-eduroam** role installs and configures FreeRADIUS for eduroam EAP authentication. In the context of this IMAP playbook, it configures it to use PAM as an authentication backend.

The configuration (and the structure of the role) largely follow the GÉANT documentation for [eduroam service providers](https://wiki.geant.org/display/H2eduroam/freeradius-sp) and [eduroam identity providers](https://wiki.geant.org/display/H2eduroam/freeradius-idp). There are some minor differences to reflect later versions of FreeRADIUS, and to leave a little documentation in place. In addition, the role defaults to creating configuration relevant to the [South African NRO](https://eduroam.ac.za/). However, the [documentation](roles/freeradius-eduroam/README.md) explains how to change this.

### [eapol_test](roles/eapol_test)

The **eapol_test** role builds and installs the [eapol_test](http://deployingradius.com/scripts/eapol_test/) and [rad_eap_test](https://github.com/CESNET/rad_eap_test) utilities. These are useful for testing and debugging (see below).

## Additional configuration

There are a couple of things that are not configured by the playbook and may require additional configuration.

* **IMAP SSL Certificate** -- if your IMAP server does not use a certificate that's in the trust store for your server, you may need to supply the certificate and configure [roles/pam-imap/templates/pam_imap.conf.j2](pam_imap.conf) accordingly.

* **EAP SSL Certificate** -- the role uses the default certificates created by the Ubuntu FreeRADIUS package. You may want to replace these.

## Testing

You can test the PAM component like this:

```bash
pamtester pam-imap-radius $USERNAME authenticate
```

You can test the inner EAP tunnel line this:

```bash
radtest -t pap $USERNAME@$REALM $PASSWORD localhost:18121 0 testing123
```

A complete EAP test can be done with eapol_test and rad_eap_test like this:

```bash
rad_eap_test -H localhost -P 1812 -S testing123 -u $USERNAME@$REALM -A anon
ymous@$REALM -p $PASSWORD -m WPA-EAP -s eduroam -e TTLS -2 PAP
```

## Notes & Limitations

### FreeRADIUS version

Since Ubuntu 16.04 LTS still uses [FreeRADIUS 2.2.8](https://packages.ubuntu.com/xenial/amd64/freeradius), the **freeradius-eduroam** role installs a FreeRADIUS 3.0.x series package from an [Ubuntu PPA](https://launchpad.net/~freeradius/+archive/ubuntu/stable-3.0/). An archived 2.2.x series configuration is [available in a separate branch](https://github.com/safire-ac-za/eduroam-imap-playbook/tree/freeradius-2.x) but, since 2.2.x is end-of-life, no futher work will be done on the branch.

### EAP-TTLS/PAP

Because FreeRADIUS has no access to a cleartext password when authenticating via PAM, only PAP can be used as an inner/phase two authentication method. This typically means you need to configure clients to use TTLS/PAP (PEAP/MSCHAPv2 will **not** work). This was not originally supported out-the-box by Windows, but Windows 10 now includes a TTLS supplicant. Likewise Android and wpasupplicant on Linux both support TTLS.

### Scalability

Setting up an IMAP connection is slow (~ 2 seconds) and more resource intensive than other authentication methods. This means that the IMAP approach likely does not scale! **pam_imap** does do some caching of credentials which may improve this. Nevertheless you should really only consider this for the smallest of sites.

### GSuite and 2FA

If you're using this with Google GSuite and your account has two-factor authentication turned on, you obviously can't supply the second factor via EAP. This means you won't be able to authenticate on eduroam with your normal GSuite username and password. Fortunately Google have preempted the problem in the form of [app passwords](https://support.google.com/accounts/answer/185833?hl=en), which are designed for legacy applications that don't suppport 2FA. To authenticate on eduroam using a GSuite account that has 2FA turned on, you'll need to [generate a new app password](https://security.google.com/settings/security/apppasswords) and then use your GSuite username and the app password to connect.
