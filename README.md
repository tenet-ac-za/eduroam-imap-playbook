# eduroam-imap-playbook Ansible playbook

This is an [Ansible playbook](http://docs.ansible.com/ansible/latest/playbooks.html) to set up [eduroam](https://eduroam.org/) authentication against an IMAP server.

Our main use-case for this is allowing [Google GSuite](https://gsuite.google.com/) users to authenticate on eduroam. However, the concept should be extensible to any IMAP server.

## Using the playbook

All configuration happens in [group_vars/all](group_vars/all). You should edit that file to suit your enviroment.

The playbook has been set up to be readily used in a [Vagrant](https://www.vagrantup.com/) enviroment with [VirtualBox](https://www.virtualbox.org/) as the provider. In such an enviroment, the following should get you going:

```bash
git clone https://github.ac.za/safire-ac-za/eduroam-imap-playbook.git
cd eduroam-imap-playbook
vagrant up
vagrant ssh eduroam-imap
```
In other enviroments, the [Vagrantfile](Vagrantfile) may need to be edited. Alternatively, the playbook should be usable without Vagrant given an appropriate [inventory](inventories/).

The playbook is also (hopefully) self documenting, allowing people who don't make use of Ansible to follow the configuration.

## Roles

This playbook contains two roles, both of which should be usable independently and outside of the playbook.

### [pam-imap](roles/pam-imap)

The **pam-imap** role downloads, compiles, installs, and configures the [pam-imap](https://github.com/wdoekes/pam-imap) pluggable authentication module. This PAM module allows for authentication against an IMAP server. By default, this is configured to be `imap.gmail.com`. See the [documentation](roles/pam-imap/README.md) for how to change this.

### [freeradius-eduroam-pam](roles/freeradius-eduroam-pam)

The **freeradius-eduroam-pam** role installs and configures FreeRADIUS to make use PAM for eduroam EAP authentication. 

The PAM-specific bits of the configuration are quite limited, and can be disabled by setting `use_pam: no`. This allows the role to be re-used more generically to set up an eduroam IdP.

The configuration (and the structure of the role) largely follow the GÉANT documentation for [eduroam service providers](https://wiki.geant.org/display/H2eduroam/freeradius-sp) and [eduroam identity providers](https://wiki.geant.org/display/H2eduroam/freeradius-idp). There are some minor differences to reflect later versions of FreeRADIUS, and to leave a little documentation in place. In addition, the role defaults to creating configuration relevant to the [South African NRO](https://eduroam.ac.za/). However, the [documentation](roles/freeradius-eduroam-pam/README.md) explains how to change this.

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

A complete EAP test can be done with [eapol_test](http://deployingradius.com/scripts/eapol_test/) and [rad_eap_test](https://github.com/safire-ac-za/rad_eap_test) as below. However neither of these two applications are installed by this Ansible role.

```bash
rad_eap_test -H localhost -P 1812 -S testing123 -u $USERNAME@$REALM -A anon
ymous@$REALM -p $PASSWORD -m WPA-EAP -s eduroam -e TTLS -2 PAP
```

## Limitations

### EAP-TTLS/PAP

Because FreeRADIUS has no access to a cleartext password, only PAP can be used as an inner/phase two authentication method. This typically means you need to configure clients to use TTLS/PAP. This was not originally supported out-the-box by Windows, but Windows 10 now includes a TTLS supplicant.

### Scalability

Setting up an IMAP connection is slow (~ 2 seconds) and more resource intensive than other authentication methods. This means that the IMAP approach likely does not scale! *pam_imap* does do some caching of credentials which may improve this. Nevertheless you should really only consider this for the smallest of sites.
