# Iptables (Ansible Role)

This role applies a strict, secure `iptables` set of rules with many configurable options. 

Default rules are:

- allow all loopback traffic
- block invalid packets, TCP portscan packets
- block all traffic involving bogons (martians)
- block deprecated ICMP messages
- log all new accepted inbound, dropped inbound, accepted outbound and dropped outbound traffic seperately

Note that no other traffic is allowed inbound/outbound by default, if you want specific ports opened you will have to specify that yourself.

In addition to this, a package to persist `iptables` rules upon reboot is installed and configured, and if bogon or host banning is enabled, `ipset` is installed.
A script is also installed that persists `ipset` sets upon reboot as well.

The benefits of using this role are robust logging, portscan blocking, automatic host banning, ICMP filtering (ICMP echo shells shouldn't work), and strict rules that only allow established traffic outbound for an inbound rule and vice versa, as well as filtering outbound traffic by specific user/groups.

## Requirements

This role uses the [iptables-raw](https://github.com/Nordeus/ansible_iptables_raw) module for manipulating `iptables`. The embedded module is current as of commit https://github.com/Nordeus/ansible_iptables_raw/commit/86ee3e0997af235bcd0a8b1ae5982f43e9612518.

## Role Variables

|Name|Default Value|Description|
|----|-------------|-----------|
|`iptables_keep_unmanaged_rules`|no|'yes' if you want to keep all rules and chains not managed by this Ansible role|
|`ipset_destroy_sets`|false|true if you want to destroy all previously created `ipset` sets|
|`iptables_log_limit`|1/sec|amount of log messages from `iptables` that will be printed. Useful for supressing multiple logs of similar events|
|`iptables_log_level`|info|level at which `iptables` will log|
|`iptables_global_allow`|[]|IPs or ranges to always allow|
|`iptables_global_block`|[]|IPs or ranges to always block|
|`iptables_ban_hosts`|false|if true, rules will be created that automatically ban hosts that trigger a threshold of DROP rules within a timeframe|
|`iptables_ban_interval`|5/hour|amount of DROP rules a host can hit within a timeframe that will trigger a ban|
|`iptables_ban_burst`|5|the number of initial packets to allow before starting to match on the ban interval|
|`iptables_ban_expire`|10800000|amount of milliseconds to remember hosts that triggered DROP rules|
|`iptables_ban_duration`|600|amount of seconds to ban offending hosts|
|`iptables_block_attacks`|true|true if you want to block invalid and portscanning TCP packets|
|`iptables_allow_ping_inbound`|false|true if you want to make your box pingable|
|`iptables_allow_ping_outbound`|false|true if you want to ping other boxes|
|`iptables_ping_users`|root|users to allow sending ICMP echo-requests outbound|
|`iptables_filter_icmp`|true|true if you want to block deprecated ICMP messages|
|`iptables_block_bogons`|true|true if you want to block all traffic involving bogons/martians|
|`iptables_allow_inbound`|[]|additional ports to allow inbound|
|`iptables_allow_outbound`|[]|additional ports to allow outbound|
|`iptables_configuration_enabled`|true|true if you want this role to run|

## Outbound Ports

Up to 15 ports can be specified for outbound rules. Ports must be separated with a comma with no whitespace in between, ex. `80,443`. Port ranges can be specified by entering a colon between two ports, ex. `1024:65535`. This counts as two out of the 15 max ports.

## Users Filering

Any of the `*_users` variables can be set to an empty list to make them allow all users.

## Additional Rules

The `iptables_allow_inbound` and `iptables_allow_outbound` variables can be used to add additional rules. Both varibles take a list of dicts, with the following keys:

- name: name of the service
- state: optional; 'present' if you want to enable the rule, and 'absent' if you want to disable it
- proto: the protocol to allow
- port: the port to allow for inbound rules
- ports: the port(s) to allow for outbound rules, up to 15 ports can be specified
- users: optional; the users to allow
- allowed: optional; IPs to allow
- blocked: optional; IPs to block

## ICMP Filtering

From the guidance of https://tools.ietf.org/pdf/draft-ietf-opsec-icmp-filtering-04.pdf, 5 deprecated/unused ICMP messages/message types are blocked by default via the `iptables_filter_icmp` variable:

- `protocol-unreachable`
- `source-quench`
- all `redirect` messages
- `timestamp` request and reply
- `address-mask` request and reply

Additionally, `echo` request and reply messages are blocked by default but can be enabled with the `iptables_allow_ping_inbound` and `iptables_allow_ping_outbound` variables.

## Host Banning (Expirimental)

There is an expirimental option to ban hosts that send a certain number of dropped packets inbound. By default hosts are banned for 10 minutes. This option is disabled by default as I have not been able to get it working 100% reliably. If I have a SSH session open on a host that has host banning enabled, and the session gets interupted, I am sometimes blocked. 

When I figure out why this is and make host banning more reliable, I'll make this option enabled by default.

## IPv6

Currently, this role drops all IPv6 packets. If I need to use IPv6 in the future, I'll work to make this role work with IPv6.

## Installation

Install the role with ansible-galaxy:

```shell
ansible-galaxy install capnspacehook.iptables
```

## Example Playbook

```yaml
- hosts: localhost
  roles:
    - capnspacehook.iptables
  vars:
    iptables_allow_inbound:
      - name: postgres
        port: 5432
        proto: tcp
        users: 
          - postgres
``` 
