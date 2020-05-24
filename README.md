# Iptables (Ansible Role)

This role applies a strict, secure `iptables` set of rules with many configurable options. 

Default rules are:

- allow all loopback traffic
- block invalid packets, TCP portscan packets
- block all traffic involving bogons (martians)
- allow SSH inbound
- allow DNS, HTTP, and HTTPS outbound for updating
- allow NTP outbound
- allow ICMP echo requests inbound and ICMP replies outbound
- ban hosts that trigger multiple DROP rules in a configurable timeframe
- log all new accepted inbound, dropped inbound, accepted outbound and dropped outbound traffic seperately

In addition to this, a package to persist `iptables` rules upon reboot is installed and configured, and if bogon or host banning is enabled, `ipset` is installed.

The benefits of using this role are robust logging, portscan blocking, automatic host banning, ICMP rate limiting and filtering (most ICMP reverse shells shouldn't work), and strict rules that only allow established traffic outbound for an inbound rule and vice versa, as well as filtering outbound traffic by specific user/groups.

## Requirements

This role uses the [iptables-raw](https://github.com/Nordeus/ansible_iptables_raw) module for manipulating `iptables`. The embedded module is current as of commit https://github.com/Nordeus/ansible_iptables_raw/commit/86ee3e0997af235bcd0a8b1ae5982f43e9612518.

## Role Variables

|Name|Default Value|Description|
|----|-------------|-----------|
|`iptables_clear_rules`|true|true if you want to flush all previously created rules and chains|
|`ipset_destroy_sets`|true|true if you want to destroy all previously created `ipset` sets|
|`iptables_log_limit`|5/min|amount of log messages from `iptables` that will be printed. Useful for supressing multiple logs of similar events|
|`iptables_log_level`|info|level at which `iptables` will log|
|`iptables_ban_hosts`|true|if true, rules will be created that automatically ban hosts that trigger a threshold of DROP rules within a timeframe|
|`iptables_ban_interval`|5/hour|amount of DROP rules a host can hit within a timeframe that will trigger a ban|
|`iptables_ban_burst`|5|the number of initial packets to allow before starting to match on the ban interval|
|`iptables_ban_expire`|10800000|amount of milliseconds to remember hosts that triggered DROP rules|
|`iptables_ban_duration`|600|amount of seconds to ban offending hosts|
|`iptables_allow_ssh`|true|true if you want to allow SSH inbound|
|`iptables_ssh_port`|22|port to allow SSH inbound on|
|`iptables_ssh_user`|root|user that owns the SSH daemon and/or that should be able to make SSH connections|
|`iptables_allow_updates`|true|true if you want to allow ports `53 udp`, `80 tcp` and `443 tcp` outbound|
|`iptables_allow_ntp`|true|true if you want to allow port `123 udp` outbound|
|`iptables_ntp_port`|123|port to allow NTP inbound on|
|`iptables_ntp_user`|root|user that owns the NTP daemon, for example if `chrony` is installed this should `_chrony`|
|`iptables_allow_ping`|true|true if you want to allow ICMP echo requests inbound|
|`iptables_ping_user`|root|user to allow ICMP replies outbound|
|`iptables_block_bogons`|true|true if you want to block all traffic involving bogons/martians|
|`iptables_allow_inbound`|[]|additional ports to allow inbound|
|`iptables_allow_outbound`|[]|additional ports to allow outbound|
|`iptables_configuration_enabled`|true|true if you want this role to run|

## User Filering

Any of the `*_user` variables can be unset or set to an empty string to make them allow all users.

## Additional Rules

The `iptables_allow_inbound` and `iptables_allow_outbound` variables can be used to add additional rules. Both varibles take a list of dicts, with the following keys:

- name: optional; name of the service
- proto: the protocol to allow
- port: the port to allow
- user: optional; the user to allow

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
    iptables_log_limit: 1/sec
    iptables_ntp_user: _chrony
    iptables_allow_inbound:
      - name: leet_server
        port: 1337
        proto: udp
        user: root
``` 
