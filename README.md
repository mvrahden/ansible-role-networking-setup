# ansible-role-networking-setup <!-- omit in toc -->

An [Ansible](https://www.ansible.com) role to define static IPs, set hostnames and setup iptables on debian-based systems.

## Table of Contents <!-- omit in toc -->

- [Side notes](#Side-notes)
- [Role Variables](#Role-Variables)
  - [Networking Interface Profiles](#Networking-Interface-Profiles)
  - [iptable definitions](#iptable-definitions)
  - [Sample Roles](#Sample-Roles)
    - [Network interfaces and Static IPs](#Network-interfaces-and-Static-IPs)
    - [Hostname](#Hostname)
- [Dependencies](#Dependencies)
  - [Requirements](#Requirements)
- [License](#License)
- [Author Information](#Author-Information)

## Side notes

This package provides 3 "subtask" functionalities.
Each functionality can be addressed by adding it to the `subtasks`-list (e.g. `subtasks: [ 'interfaces' ]`).
The functionality values are as follows:

- `'interfaces'` - sets the network interface definitions based on given **interface profiles**, hence e.g. static IPs, dns-server, routing etc.
- `'hostname'` - sets the hostname, and statically cross-references all other hosts from the `networking_group_name` via the `/etc/hosts`-file
- `'netfilter'` - sets iptables definitions

## Role Variables

[defaults/main.yml](defaults/main.yml)

### Networking Interface Profiles

This is a list of interface profile definitions, which will be translated into the requirements of the default renderer for network configuration of each operating system, such as:

- dhcpcd
- NetworkManager
- Netplan

This is the structure of the profile definitions:

```yaml
# profiles
networking_interface_profiles:
  - type: iface
    interface: lo
    address_family: inet      # or inet6 or ipx
    address_method: loopback  # or dhcp
    auto: lo                  # optional; manpage interfaces(5)
  - type: iface
    interface: eth0
    address_family: inet      # or inet6 or ipx
    address_method: static    # or dhcp
    auto: eth0
    allow_hotplug: eth0
    cidr_notation: 192.168.1.150/24 # static ip as CIDR
    address: 192.168.1.150          # static ip
    netmask: 255.255.255.0  # results into CIDR suffix /24
    gateway: 192.168.1.1
    network: 192.168.1.0
    broadcast: 192.168.1.255
    dns_nameservers: # are there any local DNS Name Servers?
      - 192.168.1.100
      - 192.168.1.1
      - 8.8.8.8
```

### iptable definitions

```yaml
networking_iptables_definitions:
  - chain: FORWARD
    ctstate: RELATED,ESTABLISHED
    in_interface: wlan0
    jump: ACCEPT
    out_interface: eth0
    state: present
    table: filter
  - chain: FORWARD
    in_interface: eth0
    jump: ACCEPT
    out_interface: wlan0
    state: present
    table: filter
  - chain: POSTROUTING
    jump: MASQUERADE
    out_interface: wlan0
    source: 192.168.1.0/24
    state: present
    table: nat
```

### Sample Roles

Assuming you have the variable `networking_interface_profiles` defined.

#### Network interfaces and Static IPs

Setup static IPs as follows:

> Attention: This Subtasks ends with a Reboot!

```yaml
  roles:
    - role: mvrahden.networking-setup
      subtasks: [ 'interfaces' ]
      networking_interface_profiles: "{{ my_awesome_interface_profiles }}"
```

#### Hostname

Setup hostnames as follows:

```yaml
  roles:
    - role: mvrahden.networking-setup
      subtasks: [ 'hostname' ]
      networking_group_name: my_clustered_hosts # inventory name
      networking_group_domain: example.com
      networking_device_interfaces: "{{ my_awesome_interface_profiles }}"
```

## Dependencies

None

### Requirements

Packages installed on your system:

- [Ansible](https://www.ansible.com)

## License

MIT

## Author Information

- Menno van Rahden
