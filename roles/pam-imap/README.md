pam-imap Ansible role
=====================

Download, compile and install the pam_imap PAM module on Ubuntu

Requirements
------------

This role is designed to be played on Ubuntu. It has been tested with Ubuntu 16.04 LTS.

Role Variables
--------------

 * `pam_imap_version` - the release version we should install
 * `pam_service_name` - the name of the PAM service we should install (defaults to `imap-auth`)
 * `imap_server` - the hostname or IP of the IMAP server (defaults to `imap.gmail.com`)
 * `imap_port` - the port number the IMAP server runs on (defaults to `993`)
 * `imap_ssl` - whether the IMAP server requires SSL-on-connect (defaults to `yes`)
 * `imap_realm` - the realm/domain your IMAP server uses for usernames (defaults to none)
 * `imap_cache` - how long (in seconds) to cache IMAP credentials for (defaults to 600)

Dependencies
------------

This role is intended to be used in conjunction with the freeradius-pam-eduroam role.

Example Playbook
----------------

See <https://github.com/safire-ac-za/eduroam-imap-playbook/>

License
-------

MIT <https://github.com/safire-ac-za/eduroam-imap-playbook/LICENSE>

Author Information
------------------

Guy Halse <http://orcid.org/0000-0002-9388-8592>,
[Tertiary Education and Research Network of South Africa](http://www.tenet.ac.za/).
