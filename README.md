OpenWrt Knot DNS
================

This role configures [Knot DNS](https://www.knot-dns.cz/) as an authoritative name server on [OpenWrt](https://www.openwrt.org/) targets.

Requirements
------------

This role requires the `ansible.utils` collection and the [netaddr](https://github.com/netaddr/netaddr/) Python package on the Ansible controller.

Moreover, it requires a working [Python](https://www.python.org/) installation on the target system or [gekmihesg's Ansible library for OpenWrt](https://github.com/gekmihesg/ansible-openwrt) on the Ansible controller.

Role Variables
--------------

* `knot_config`  
  A dictionary of Knot DNS configuration options to set.
  Refer to the [Knot DNS documentation](https://www.knot-dns.cz/documentation/) for options and their meaning.
  It is not necessary to add entires to the `zone` section for the zones defined in `knot_zones`.
  This role automatically adds minimal entries.
  However, it is possible to add entries to set zone-specific options.
  When doing so, it is important to either use the `ansible` template or otherwise set the options that are part of the `ansible` template.
  Without these options, this role can not reliably handle zone data updates.
  Furthermore, it is possible to modify the `ansible` template, simply by adding an entry with the id `ansible` to the `template` section.
  This role automatically ensures that this template contains all required options, so it is not necessary to set them manually.
  Optional.
* `knot_zones`  
  A dictionary containing the zone contents that Knot DNS should server.
  Dictionary keys are zone names and value are in turn dictionaries with the following keys:
  * `ttl`  
    The zone's default TTL (time-to-live).
    It is used for records that do not explicitly set a TTL.
    Optional.
  * `records`  
    A list of records to serve.
    Each list item defines a DNS record and may be either a valid line for a zone file (cf. RFC 1035 Section 5, RFC 1034 Section 3.6.1) or a dictionary with the following keys:
    * `name`  
      The relative or absolute name of the record.
      Mandatory.
    * `ttl`  
      The TTL value for this record.
      Optional.
    * `class`  
      The record's class.
      Defaults to `IN` (Internet).
    * `type`  
      The record's type.
      Mandatory, except when the `data` file contains an IPv4 or IPv6 address.
    * `data`  
      The record's data.
      May either be a single value or a list to define a RRset with multiple values.
      Mandatory.
* `knot_zones_dir`  
  The directory where the zone files for all zones managed by this role are stored.
  Defaults to `/etc/knot/zones/`.

Dependencies
------------

This role does not depend on any specific roles.

Example Configuration
---------------------

The following is a short example for some of the configuration options this role provides:

```yaml
knot_config:
  server:
    listen:
      - 0.0.0.0@53
      - ::@53
  policy:
    - id: automatic
      algorithm: ed25519
      zsk-lifetime: 60d
      cds-cdnskey-publish: rollover
  template:
    - id: ansible
      dnssec-signing: true
      dnssec-policy: automatic
  zone:
    - name: example.org
      dnssec-singing: false
knot_zones:
  example.com:
    ttl: 3600
    records:
      - name: '@'
        ttl: 86400
        type: SOA
        data: "ns.icann.org. noc.dns.icann.org. 1 2H 1H 14D 1H"
      - name: '@'
        ttl: 72000
        type: NS
        data:
          - a.iana-servers.net.
          - b.iana-servers.net.
      - 'host1 IN A 192.0.2.1'
      - 'host1 IN A 192.0.2.2'
      - name: host2
        data:
          - 192.0.2.3
          - 192.0.2.4
  example.org:
    records:
      - name: '@'
        ttl: 86400
        type: SOA
        data: "ns admin 1 1D 2H 2D 1H"
```

Note on DNSSEC
--------------

When configured to do so, Knot DNS handles most aspects of DNSSEC (key generation, signing, key rollover, ...) automatically.
However, it can not handle adding the correct `DS` record to the parent zone.
As the process to do so varies widely, this role does not handle it either and leaves that task up to the DNS operator.

License
-------

MIT
