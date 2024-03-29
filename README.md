# Ansible Role: Firewall

An Ansible Role that installs iptables and configure IPv4 & IPv6 Rules.

We want this role as simple, configurable and interoperable as possible.

## Requirements

Compliant with :
- Debian 11 (Bullseye)
- Debian 10 (Buster)
- Debian 9 (Stretch)
- Debian 8 (Jessie)
- Ubuntu 20 (Focal Fossa)
- CentOS 8 (WIP)

Other:
- Fact gathering should be allowed in ansible-playbook (By default)
- The role require the superuser privileges. The task should be used with remote_user root or with a sudo/su grant
- ansible builtin service_facts must be populated

## Role behavior

- Disable other firewalls (Optionnal)
- Install required packages
- Deploy configuration for packets logging (Optionnal)
- Generate iptables rules for IPv4 & IPv6
- Test and restore respectively IPv4 & IPv6 rules only if :
        ( Rules have been updated ) or ( Current and generated rules differ )

## Role Variables

Available variables with defaults values are defined in `defaults/main.yml`.

## Dependencies

none

## Rules Syntax

### Filter Rules & Nat Rules & Roles Rules syntax

Here is a recap of all possible values for firewall_rules & for role rules.

| Option    | Role                            | Default value | Possible values                     |
|-----------|---------------------------------|:-------------:|-------------------------------------|
| table     | Table                           |     filter    | filter, nat                         |
| in_if     | Input Interface (-i)            |      any      | Any interface name available        |
| out_if    | Output Interface (-o)           |      any      | Any interface name available        |
| ip        | IP Version                      |      both     | 4, 6                                |
| chain     | iptables chain                  |     INPUT     | INPUT, FORWARD, OUTPUT              |
| proto     | Transport Protocol              |      none     | tcp, udp, vrrp, etc.                |
| match     | Match                           |      proto    | null (disable), tcp, udp, etc.      |
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
    { in_if: 'eth1', proto: 'udp', src_ip: '23.66.165.166/32', dest_ip: '104.16.77.187', dest_port: '1194' },
    { ip: '6', proto: 'udp', dest_ip: '2001:4860:4860::8888', dest_port: '53' }
    { dest_port: '8484' },
]
```

This will be respectively translated to :

```
IPv4 :
-A INPUT -p tcp -m tcp --dport 80 -m comment --comment "Accept HTTP" -j ACCEPT
-A INPUT -i eth1 -p udp -m udp --source 23.66.165.166/32 --destination 104.16.77.187/32 --dport 1194 -j ACCEPT
-A INPUT --dport 8484 -j ACCEPT

IPv6 :
-A INPUT -p tcp -m tcp --dport 80 -m comment --comment "Accept HTTP" -j ACCEPT
-A INPUT -p udp -m udp --destination 2001:4860:4860::8888/128 --dport 53 -j ACCEPT
-A INPUT --dport 8484 -j ACCEPT
```

### Forwarded Ports Rules syntax

Possible values for firewall_forwarded_ports_rules.

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

If you use another role called "nginx", simply add a specific variable for it to the firewall_enabled_role_rules_list. You can choose the name you want but choose wisely, something unique, we will set it as fact for using it in the other role firewall. We don't want to override another variable.

```
firewall_enabled_role_rules_list:
  - [...]
  - 'docker_firewall_rules'
  - 'nginx_firewall_rules'
```

Then, in your "nginx" role, define a new fact.

```
- name: Export nginx_firewall_rules to a 'host-fact' type variable
  set_fact:
    nginx_firewall_rules: [
        { proto: 'tcp', src_ip: 'any', src_port: 'any', dest_ip: 'any', dest_port: '80', policy: 'ACCEPT' },
        { proto: 'tcp', dest_port: '443' },
    ]
```

**Note :** If you want to use tags to run only the firewall role, please use the `tags: always` for the set_fact in other roles. This ensures that even if there is only the firewall role that is executed you will not forget rules of other roles.

## License

GPLv3

## Author Information

This role is maintained by maximiliend. Issues and Merge Request are welcome.
