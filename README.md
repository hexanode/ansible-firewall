# Ansible Role: Firewall

An Ansible Role that installs iptables and configure IPv4 & IPv6 Rules.


## Requirements

Compliant with :
- Debian 8 (Jessie)
- Debian 9 (Stretch)

Other:
- Fact gathering should be allowed in ansible-playbook (By default)
- The role require the superuser privileges. The task should be used with remote_user root or with a sudo/su grant


## Role Variables

Available variables with defaults values are defined in `defaults/main.yml`.

Modifiables variables and possible values are listed below :

```
# Set to true in order to disable others firewall. (default is false)
firewall_disable_others: false

# Set false to disallow icmp. (default is true)
firewall_allow_icmp: true

# Set false to stop logging dropped / reject packets. (default is true)
firewall_log_dropped_packets: true

# Set false to reject instead of dropping packets. (default is true)
firewall_drop_instead_of_reject: true

# Set true to configure default INPUT policy to ACCEPT. (default is false)
firewall_input_policy_accept: false

# Set true to configure default FORWARD policy to ACCEPT. (default is false)
firewall_forward_policy_accept: false

# Set false to configure default OUTPUT policy to REJECT/DROP. (default is true)
firewall_output_policy_accept: true

# List of allowed tcp ports. (default is to allow ssh to preserve ssh access)
firewall_allowed_tcp_ports:
  - "22"

# List of allowed udp ports.
firewall_allowed_udp_ports: []

# List of forwarded tcp ports.
firewall_forwarded_tcp_ports: []

# List of forwarded udp ports.
firewall_forwarded_udp_ports: []

# Additional IPv4 nat firewall rules.
firewall_ipv4_additional_nat_rules: []

# Additionnal IPv6 nat firewall rules.
firewall_ipv6_additional_nat_rules: []

# Additional IPv4 firewall rules.
firewall_ipv4_additional_rules: []

# Additionnal IPv6 firewall rules.
firewall_ipv6_additional_rules: []

# Enabled Role rules list
enabled_role_rules_list: []
```

## Dependencies

none


## Rules Syntax

Here is a recap of all possible values.

| Option    | Role                            | Default value | Possible values                     |
|-----------|---------------------------------|:-------------:|-------------------------------------|
| ip        | IP Version                      |       4       | 4, 6                                |
| proto     | Transport Protocol              |      tcp      | tcp, udp                            |
| src_ip    | Specific source IP Address      |      any      | any IPv4 or IPv6 (must set ip to 6) |
| src_port  | Specific source port            |      any      | any port number                     |
| dest_ip   | Specific destination IP Address |      any      | any IPv4 or IPv6 (must set ip to 6) |
| dest_port | Specific destination port       |      any      | any port number                     |
| policy    | Policy to apply                 |     ACCEPT    | ACCEPT, DENY, DROP                  |

The syntax is a bit flexible, per example you can use the following syntax.

```
[
    { proto: 'tcp', src_ip: 'any', src_port: 'any', dest_ip: 'any', dest_port: '80', policy: 'ACCEPT' },
    { proto: 'udp', src_ip: '23.66.165.166', dest_ip: '104.16.77.187', dest_port: '1194' },
    { ip: '6', proto: 'udp', dest_ip: '2001:4860:4860::8888', dest_port: '53' }
    { dest_port: '443' },
]
```

This will be respectively translated to :

```
IPv4 :
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
-A INPUT -p udp -m udp --source 23.66.165.166 --destination 104.16.77.187 --dport 1194 -j ACCEPT

IPv6 :
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
-A INPUT -p udp -m udp --destination 2001:4860:4860::8888 --dport 53 -j ACCEPT
```

## Example Playbook

How to use the role in your ansible playbook.

```
- name: Playbook Task to install firewall role
  hosts: all (Group of servers on which you want to install and configure iptables firewalls)
  roles:
    - { role: firewall, tags: [ 'firewall' ] }
```

## Example of external role using firewall role

How to use the iptables rules creation with firewall role in a external role.

If you use another role called "nginx", simply add a specific variable for it to the enabled_role_rules_list. You can choose the name you want but choose wisely, something unique, we will set it as fact for using it in the other role firewall. We don't want to override another variable.

```
enabled_role_rules_list:
  - [...]
  - 'nginx_firewall_rules'
```

Then, in your "nginx" role, define a new fact.

```
- name: Export nginx_firewall_rules to a 'host-fact' type variable
  set_fact:
    nginx_firewall_rules: [
        { proto: 'tcp', src_ip: 'any', src_port: 'any', dest_ip: 'any', dest_port: '80', policy: 'ACCEPT' },
        { dest_port: '443' },
    ]
```

## ToDo

- Add tests
- Add CI Integration


## License

GPLv3


## Author Information

This role is maintained by maximiliend. Issues and Merge Request are welcome.
