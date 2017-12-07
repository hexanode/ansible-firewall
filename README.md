# Ansible Role: Firewall

An Ansible Role that installs iptables and configure IPv4 & IPv6 Rules.

We want this role as simple, configurable and interoperable as possible.

## Requirements

Compliant with :
- Debian 8 (Jessie)
- Debian 9 (Stretch)

Other:
- Fact gathering should be allowed in ansible-playbook (By default)
- The role require the superuser privileges. The task should be used with remote_user root or with a sudo/su grant

## Role behavior

- Disable other firewalls (Optionnal)
- Install required packages
- Deploy configuration for packets logging (Optionnal)
- Generate iptables rules for IPv4 & IPv6
- Test and restore respectively IPv4 & IPv6 rules only if :
        ( Rules have been updated ) or ( Current and generated rules differ )

## Role Variables

Available variables with defaults values are defined in `defaults/main.yml`.

Modifiables variables and possible values are listed below :

```
# Set to true in order to disable others firewall. (default is false)
firewall_disable_others: false

# Set to false to disable automatic rules restoration, the role will only generate rules.v[4|6] files. (default is true)
firewall_restore_rules: true

# Set false to disallow icmp. (default is true)
firewall_allow_icmp: true

# Set false to stop logging dropped / reject packets. (default is true)
firewall_log_dropped_packets: true

# Log at most X packets per minute. (Default is 10)
firewall_limit_log: 10

# Set false to reject instead of dropping packets. (default is true)
firewall_drop_instead_of_reject: true

# Set true to configure default INPUT policy to ACCEPT. (default is false)
firewall_input_policy_accept: false

# Set true to configure default FORWARD policy to ACCEPT. (default is false)
firewall_forward_policy_accept: false

# Set false to configure default OUTPUT policy to REJECT/DROP. (default is true)
firewall_output_policy_accept: true

# List of enabled rules. (default is to allow ssh to preserve ssh access)
firewall_filter_rules:
  - { dest_port: '22' }

# List of forwarded rules.
firewall_forwarded_rules: []

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

### Filter Rules & Roles Rules syntax

Here is a recap of all possible values for firewall_filter_rules & for role rules.

| Option    | Role                            | Default value | Possible values                     |
|-----------|---------------------------------|:-------------:|-------------------------------------|
| ip        | IP Version                      |       4       | 4, 6                                |
| proto     | Transport Protocol              |      tcp      | tcp, udp                            |
| src_ip    | Specific source IP Address      |      any      | any IPv4 or IPv6 (must set ip to 6) |
| src_port  | Specific source port            |      any      | any port number                     |
| dest_ip   | Specific destination IP Address |      any      | any IPv4 or IPv6 (must set ip to 6) |
| dest_port | Specific destination port       |      any      | any port number                     |
| policy    | Policy to apply                 |     ACCEPT    | ACCEPT, DENY, DROP                  |
| comment   | Comment                         |               | String                              |

**Note :**
- If you omit an option the default will be used.
- For src_ip & dest_ip, you can use the CIDR notation X.X.X.X/YY or a single IP X.X.X.X.

The syntax is a bit flexible, per example you can use the following syntax.

```
[
    { proto: 'tcp', src_ip: 'any', src_port: 'any', dest_ip: 'any', dest_port: '80', policy: 'ACCEPT', comment: 'Accept HTTP' },
    { proto: 'udp', src_ip: '23.66.165.166/32', dest_ip: '104.16.77.187', dest_port: '1194' },
    { ip: '6', proto: 'udp', dest_ip: '2001:4860:4860::8888', dest_port: '53' }
    { dest_port: '443' },
]
```

This will be respectively translated to :

```
IPv4 :
-A INPUT -p tcp -m tcp --dport 80 -m comment --comment "Accept HTTP" -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
-A INPUT -p udp -m udp --source 23.66.165.166/32 --destination 104.16.77.187/32 --dport 1194 -j ACCEPT

IPv6 :
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
-A INPUT -p udp -m udp --destination 2001:4860:4860::8888/128 --dport 53 -j ACCEPT
```

### Forwarded Rules syntax

Possible values for firewall_forwarded_rules.

| Option    | Role                            | Default value | Possible values                     |
|-----------|---------------------------------|:-------------:|-------------------------------------|
| ip        | IP Version                      |       4       | 4, 6                                |
| proto     | Transport Protocol              |      tcp      | tcp, udp                            |
| src_port  | Specific source port            | no (Required) | any port number                     |
| dest_port | Specific destination port       | no (Required) | any port number                     |

The syntax is a bit flexible, per example you can use the following syntax.

```
[
    { src_port: '80', dest_port: '8080' },
    { proto: 'udp', src_port: '53', dest_port: '5353' }
]
```

This will be respectively translated to :

```
IPv4 :
-A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
-A OUTPUT -p tcp -o lo --dport 80 -j REDIRECT --to-port 8080
-A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353
-A OUTPUT -p udp -o lo --dport 53 -j REDIRECT --to-port 5353

IPv6 :
-A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
-A OUTPUT -p tcp -o lo --dport 80 -j REDIRECT --to-port 8080
-A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353
-A OUTPUT -p udp -o lo --dport 53 -j REDIRECT --to-port 5353
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
- Improve current and generated rules comparison


## License

GPLv3


## Author Information

This role is maintained by maximiliend. Issues and Merge Request are welcome.
