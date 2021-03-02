ansible-freeradius3
=========

This role will install FreeRADIUS 3.0.x (currently 3.0.15). The default role variables will result in the official default FreeRADIUS installation.  
Role is flexible, so you can setup your own virtual server, OTP REST API client, LDAP modules, etc. using already prepared variables.

Requirements
------------

This role is tested with the Ansible 2.3 version.

Role Variables
--------------
Default role variables are stored in the `defaults\main.yml`. Variables are divided into blocks listed below:

`freeradius3_users` - variables for the Radius users.conf template  
`freeradius3_clients` - variables for the Radius clients.conf template  
`freeradius3_virtualservers` - virtual server definition for the sites-available/default file template  
`freeradius3_plugins` - freeradius plugins installed  
`freeradius3_mods_enabled` - mods linked to the mods-enabled directory
`freeradius3_radiusd_log` - radius logging setting
`freeradius3_rest` - defines REST API settings
`freeradius3_ldap_*` - defines Radius LDAP module variables
`freeradius3_perl_*` - defines perl module configuration

For currently supported variables, please check `defaults/main.yml`.

Example setup for the FreeRADIUS with the LDAP groups in the response. FreeRADIUS will return LDAP-Group for a given user using ldap module.  
To leverage Privacyidea authentication, installation of the `privacyidea-radius` Ansible role is required.  
```
# clients.conf
freeradius3_clients:
  - shortname: localhost
    ipaddr: 127.0.0.1
    proto: '*'
    secret: "radius1234"
    require_message_authenticator: "no"
    nas_type: other
    limit:
        max_connections: 16
        lifetime: 0
        idle_timeout: 30
  - shortname: localhost_ipv6
    ipv6addr: ::1
    secret: "radius1234"
  - shortname: "vpn-firewall"
    ipaddr: 192.168.0.0
    netmask: 24
    secret: "radius1234"
    require_message_authenticator: "no"
    nas_type: other


freeradius3_plugins:
   - freeradius
   - freeradius-common
   - freeradius-ldap
   - freeradius-utils

# default mods enabled after FreeRADIUS installation
freeradius3_mods_enabled: [
        detail.log, detail, pap, attr_filter, expr, always, perl, ldap
    ]

### radiusd logs ###
freeradius3_radiusd_log:
    destination: "files"
    colourise: "yes"
    file: >
        {% raw %}${logdir}/radius.log{% endraw %}
    syslog_facility: "daemon"
    stripped_names: "no"
    auth: "yes"           #  Log auth requests
    auth_badpass:  "no"   #  Log passwords with the authentication requests.
    auth_goodpass: "no"   #  Log passwords with the authentication requests.

# possible vpn groups
freeradius3_vpn_groups_ldap:
    - "test"
    - "basic"
    - "admin"

# mods-enabled/ldap
freeradius3_ldap_modules:
    - name: default_ldap
      ldap_debug: '0x0028' # for basic debug, set 0x0028, for full debug 0xFFFF
      ldap_server: 'localhost'
      ldap_port: 389
      ldap_identity: 'cn=admin,dc=example,dc=org'
      ldap_pass: 'pass'
      ldap_basedn: 'dc=example,dc=org'
      user_basedn: "{% raw %}${..base_dn}{% endraw %}"
      user_filter: "{% raw %}(uid=%{%{Stripped-User-Name}:-%{User-Name}}){% endraw %}"
      client_basedn: "{% raw %}${..base_dn}{% endraw %}"
      client_filter: '(objectClass=radiusClient)'
      group_basedn: "{% raw %}${..base_dn}{% endraw %}"
      group_filter: "{% raw %}(objectClass=posixGroup){% endraw %}"
      group_membership_filter: "{% raw %}(|(member=%{control:Ldap-UserDn})(memberUid=%{%{Stripped-User-Name}:-%{User-Name}})){% endraw %}"
      group_membership_attribute: 'memberOf'
      group_name_attribute: cn
      group_cacheable_name: 'no'
      group_cacheable_dn: 'no'
      ldap_tls_enable: false # disabled by default
      ldap_tls:
          ca_file: /path/to/cacert.crt
          #ca_path  = ${certdir} # there is bug in gnutls
          certificate_file: /path/to/radius.crt
          private_key_file: /path/to/radius.key
          random_file: /dev/urandom
          require_cert: 'demand'

# users
freeradius3_users:
  - template_content: >
      DEFAULT Auth-Type := perl

# sites-enabled/default
freeradius3_sites_default:
        - name: privacyidea
          listen:
            - ipaddr: "0.0.0.0"
              port: 1812
              type: auth
              limit:
                lifetime: 0
                idle_timeout: 30
            - ipaddr: "*"
              port: 0
              type: acct
          authorize:
            - -ldap
            - if (&User-Password) {
                  update control {
                     Auth-Type := 'perl'
                  }
               }
          authenticate:
             - auth_type:
                 - name: perl
                   module: perl
          preacct:
             - ""
          accounting:
             - ""
          post_auth:
             - template_content: >
                 update {
                   &reply: += &session-state:
                 }

                 -sql

                 exec

                 remove_reply_message_if_eap

                 Post-Auth-Type REJECT {

                   -sql
                   attr_filter.access_reject

                   eap

                   remove_reply_message_if_eap
                 }
          pre_proxy:
             - ""
```

Dependencies
------------
None


Example Playbook
----------------

```
- hosts: freeradius
  sudo: yes
  roles:
    - ansible-freeradius3
```

License
-------
GPL

Author Information
------------------
Peter Gasper
