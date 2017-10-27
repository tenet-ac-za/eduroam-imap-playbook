freeradius-pam-eduroam Ansible role
===================================

Install FreeRADIUS and configure it to use PAM for EAP authentication in a way that is compatible with eduroam.

Requirements
------------

This role is designed to be played on Ubuntu. It has been tested with Ubuntu 16.04 LTS.

Users will need RADIUS detail from their own eduroam national roaming operator.

FreeRADIUS config is largely in accordance with GÉANT's documentation, available at <https://wiki.geant.org/display/H2eduroam/freeradius-idp>

Role Variables
--------------

 * `eduroam_flr_servers` - a list of eduroam federation-level RADIUS servers. For each server, the following should be provided (defaults to using the South African eduroam FLR servers):
   * `name` - hostname of the FLR server
   * `ip` - IP address of the FLR server
   * `port` - port of the FLR server
   * `secret` - the RADIUS secret negotiated with the NRO/FLR operator
 * `radius_realm` - the Realm to use for your users, typically your primary DNS domain name (defaults to `example.ac.za`)
 * `radius_test_account`
   * `username`
   * `password`
 * `pam_service_name` - the name of the PAM service we should install (defaults to `radiusd`)
 * `use_pam` - set to `no` to disable use of PAM, in which case this becomes a genericish role to make eduroam work (defaults to `yes`)

Dependencies
------------

This role is intended to be used in conjunction with the pam-imap role.

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
