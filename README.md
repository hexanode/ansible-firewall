# Ansible Role: Firewall

An Ansible Role that installs iptables and configure IPv4 & IPv6 Rules.


## Requirements

Compliant with :
- Debian 8 (Jessie)
- Debian 9 (Stretch)

Other:
- Fact gathering should be allowed in ansible-playbook (By default)
- The role require the superuser privileges. The task should be used with remote_user root or with a sudo/su grant


## Role Variables

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

```

## Dependencies

none


## Example Playbook

How to use the role in your ansible playbook.

    - name: Playbook Task to install docker role
      hosts: all (Group of servers on which you want to install and configure iptables firewalls)
      roles:
        - { role: firewall, tags: [ 'firewall' ] }


## ToDo

- Add tests
- Add CI Integration


## License

GPLv3


## Author Information

This role is maintained by maximiliend. Issues and Merge Request are welcome.
