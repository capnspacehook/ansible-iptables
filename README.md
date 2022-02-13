# iptables (Ansible Role)

This role applies a strict and secure set of rules for `iptables` with many configurable options.

You declare what inbound and outbound traffic should be allowed, and this role will take care of the rest.

In addition to allowing the traffic you specify, this role will:

- install and make `iptables` and `ipset` rules persistent
- allow all loopback traffic
- block invalid packets, TCP portscan packets
- block deprecated ICMP messages
- log all new accepted inbound, dropped inbound, accepted outbound and dropped outbound traffic separately

## Role Variables

|Name|Default Value|Description|
|----|-------------|-----------|
|`iptables_keep_unmanaged_rules`|'no'|'yes' if you want to keep all rules and chains not managed by this Ansible role|
|`ipset_destroy_sets`|false|true if you want to destroy all previously created `ipset` sets|
|`iptables_log_limit`|1/sec|amount of log messages from `iptables` that will be printed. Useful for suppressing multiple logs of similar events|
|`iptables_log_level`|info|level at which `iptables` will log|
|`iptables_block_bogons`|true|true if you want to block traffic involving bogons/martians|
|`iptables_block_attacks`|true|true if you want to block invalid and portscanning TCP packets|
|`iptables_global_allow`|[]|IPs or ranges to always allow|
|`iptables_global_block`|[]|IPs or ranges to always block|
|`iptables_allow_ping_inbound`|false|true if you want to make your box pingable|
|`iptables_allow_ping_outbound`|false|true if you want to ping other boxes|
|`iptables_ping_users`|["root"]|users to allow sending ICMP echo-requests outbound|
|`iptables_filter_icmp`|true|true if you want to block deprecated ICMP messages|
|`iptables_allow_inbound`|[]|ports to allow inbound|
|`iptables_allow_outbound`|[]|ports to allow outbound|
|`iptables_configuration_enabled`|true|true if you want this role to run|

## Allowing Inbound Traffic

`iptables_allow_inbound` takes a list of rules, which have the following options:

|Name|Default Value|Required|Description|
|----|-------------|--------|-----------|
|`name`|""|true|name of the rule. Note due to `iptables` restrictions this must be less than 15 characters|
|`state`|"present"|false|"present" if the rule should be enabled, and "absent" otherwise|
|`interface`|""|false|name of the interface traffic will be allowed through|
|`proto`|""|true|name of the protocol to allow, "tcp" or "udp"|
|`port`|0|true|port that will be allowed|
|`filter_loopback`|false|false|whether to filter inbound loopback traffic on this port or not|
|`action`|"ACCEPT"|false|the action to take on new inbound traffic|
|`inbound_est_action`|"ACCEPT"|false|the action to take on established inbound traffic|
|`outbound_est_action`|"ACCEPT"|false|the action to take on established outbound traffic|
|`block_bogons`|false|false|whether to block bogon (private) traffic or not|
|`allowed`|[]|false|IPs or ranges to allow|
|`blocked`|[]|false|IPs or ranges to block|

## Allowing Outbound Traffic

`iptables_allow_inbound` takes a list of rules, which have the following options:

|Name|Default Value|Required|Description|
|----|-------------|--------|-----------|
|`name`|""|true|name of the rule. Note due to `iptables` restrictions this must be less than 15 characters|
|`state`|"present"|false|"present" if the rule should be enabled, and "absent" otherwise|
|`interface`|""|false|name of the interface traffic will be allowed through|
|`proto`|""|true|name of the protocol to allow, "tcp" or "udp"|
|`ports`|0|true|ports that will be allowed|
|`filter_loopback`|false|false|whether to filter outbound loopback traffic on these ports or not|
|`action`|"ACCEPT"|false|the action to take on new outbound traffic|
|`inbound_est_action`|"ACCEPT"|false|the action to take on established inbound traffic|
|`outbound_est_action`|"ACCEPT"|false|the action to take on established outbound traffic|
|`filters`|[]|false|additional restrictions to apply|

### Outbound Ports

Up to 15 ports can be specified for outbound rules. Ports must be comma separated, ex. `80,443`. Port ranges can be specified by entering a colon between two ports, ex. `1024:65535`. This counts as two out of the 15 max ports.

### Outbound Filters

Outbound rules allow multiple filters to be applied per rule. This can enable different behavior for different users making outbound connections to the same port.
You can for example only allow certain users to connect to certain IPs, and allow other users to have unrestricted access on the same port.
Outbound filters have the following options:

|Name|Default Value|Required|Description|
|----|-------------|--------|-----------|
|`name`|""|true|name of the rule. Note due to `iptables` restrictions this must be less than 15 characters when combined with the name of the parent outbound rule|
|`users`|[]|false|users that this filter will match on|
|`block_bogons`|false|false|whether to block bogon (private) traffic or not|
|`allowed`|[]|false|IPs or ranges to allow|
|`blocked`|[]|false|IPs or ranges to block|
|`action`|"ACCEPT"|false|the action to take on new outbound traffic|
|`est_action`|"ACCEPT"|false|the action to take on established outbound traffic|

## ICMP Filtering

From the guidance of https://tools.ietf.org/pdf/draft-ietf-opsec-icmp-filtering-04.pdf, 5 deprecated/unused ICMP messages/message types are blocked by default via the `iptables_filter_icmp` variable:

- `protocol-unreachable`
- `source-quench`
- all `redirect` messages
- `timestamp` request and reply
- `address-mask` request and reply

Additionally, `echo` request and reply messages are blocked by default but can be enabled with the `iptables_allow_ping_inbound` and `iptables_allow_ping_outbound` variables.

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
        proto: tcp
        port: 5432
    iptables_allow_outbound:
      - name: https
        proto: tcp
        port: 443
        filters:
          - name: apt
            users:
              - _apt
          - name: metrics
            users:
              - prometheus
            allowed:
              - 192.168.1.23
``` 
