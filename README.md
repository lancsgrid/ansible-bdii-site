# ansible-bdii-site
Ansible role for a site bdii

Requirments
-----------

CC7 or compatable OS (with ssh access via public key encryption).

Role
-----------

This role will install and configure a CC7 machine to act as a site BDII for Grid purposes.  All configuration is done through ansible using user supplied variables.

Variables
-----------

Nearly all variables use the same name as specified in the SysAdmin guide here: http://gridinfo.web.cern.ch/information-system-sys-admins. These variables are listed below as well.

Quick Setup
-----------

You will need to pass several variables to this role, whilst many variables have defaults, there are a lot that are site specific. 

Required Variables
-------

For the BDII site configuration file which specifies the site attributes, these are:

| Variable | Description |
|--------- | ----------- |
|SITE_NAME | Sites full name e.g. UKI-NORTHGRID-LANCS-HEP |
|SITE_COUNTRY| Country that site is in |
|SITE_DESC| A description of the site |
|SITE_WEB| URL for sites website |
|SITE_LOC| Sites location: <City, Country> |
|SITE_LAT| Optional, site latitude |
|SITE_LONG| Optional, site longitude |
|SITE_EMAIL| Sites contact email |
|SITE_SECURITY_EMAIL| Sites security contact email |
|SITE_SUPPORT_EMAIL| Sites support email |

Additionally, and additional information for the "OTHERINFO" variable can be passed using an ansible list as so:

```
OTHERINFO:
- GRID=EGEE
- GRID=GRIDPP
- GRID=WLCG
- GRID=NORTHGRID
- TIER=2
```

You also need to provide a list of resource BDIIs that will be published in the site BDII.  The BDII host itself is automatically added to the list, a YAML dictionary of aliases, and URLs should be provided e.g.:

```
SITEURLS:
    CE1: CE1.lancs.ac.uk
    SE1: SE1.lancs.ac.uk
```

Optional Variables
-------------

The first optional variable is for whether you would like ansible to configure the firewall on this node.  If so you need to set the variable `control_iptables` to be `true`.  You will then need to provide an ip address or range using the variable `ssh_gateway`.

Additional input rules can be passed using an ansible list like so:

```
iptables_rules:
- '-A INPUT -s ganglia.lancs.ac.uk -m state --state NEW -m tcp -p tcp --dport 8649 -j ACCEPT'
```

All other variables listed in the BDII Documentation can be set using the same variable name.  The only variable you will probably want to set is for whether your site supports IPv6 (`BDII_IPV6_SUPPORT`) in which case you need to configure the firewall manually as it blocks all IPv6 connections.


Example
------------

An example playbook would be:

bdii-site.yml
```
- hosts: bdii-site
  remote_user: root
  roles:
    - ansible-bdii-site
  vars:
   SITE_NAME: UKI-NORTHGRID-LANCS-HEP-TEST
   SITE_COUNTRY: UK
   SITE_DESC: A UK Site
   SITE_WEB: https://lancsgrid.wordpress.com
   SITE_LOC: Lancaster, UK
   SITE_LAT: 54.0105
   SITE_LONG: -2.7840
   SITE_EMAIL: admin@domain.invalid
   SITE_SECURITY_EMAIL: admin@domain.invalid
   SITE_SUPPOT_EMAIL: admin@domain.invalid
   OTHERINFO: ['GRID=EGEE', 'GRID=GRIDPP', 'GRID=WLCG', 'GRID=NORTHGRID', 'TIER=2']
   SITEURLS: {CE1: CE1.lancs.ac.uk, SE1: SE1.lancs.ac.uk}
   control_iptables: true
   ssh_gateway: 0.0.0.0/0
   iptables_rules: ['-A INPUT -s ganglia.lancs.ac.uk -m state --state NEW -m tcp -p tcp --dport 8649 -j ACCEPT']
```

Which can then be ran by doing:
`ansible-playbooks -i inventory -s bdii-site.yml`  

This assumes the existance of an inventory file where the bdii server is in the group 'bdii-site'.


Author
------------
Robin Long