Ansible Role: Wireguard
=======================

Ansible role to install and configure [Wireguard](https://www.wireguard.com).

A lot of roles setup the tunnel by using `wg-quick`. It is fine in most cases,
but sometimes some automatic behiaviors of `wg-quick` are not wanted. This role
allows to setup a tunnel by using `wg-quick`, `systemd-networkd` or just
generate a wireguard configuration that can be loaded by `wg`.

Dependencies
------------

For distributions that are not providing packages for wireguard in their
official repositories, correct repositories need to be setup before using this
role.

How-to
------

Declare the wireguard profiles in `wireguard_profiles`

```yaml
wireguard_profiles:
  # Tunnel interface name
  tunnel_name:
    # How to setup the tunnel
    # Possible values are:
    #   wg-quick: using wg-quick
    #   systemd-networkd: generate a .netdev file to create the tunnel
    #   wireguard-conf: generate a wireguard configuration that can be used with `wg`
    method:

    # MTU variable, in bytes. Used by wg-quick and systemd-networkd. Optional
    mtu:

    # Listen port. Optional
    listen_port:

    private_key:
    fwmark:

    # String to dump at the end of the .netdev or configuration file, depending
    # on the method. Can be used to add additional options.
    additional_opts:

    ## Used by wg-quick only ##

    # Enable or not the tunnel
    enable: True
    # Addresses to set on the tunnel interface. Optional
    addresses: []
    # DNS addresses to use. Optional
    dns: []
    # Route table to use. Optional
    table:
    # Post up script. Optional
    post_up:
    # Post down script. Optional
    post_down:

    ###########################

    peers:
        # Description of the peer. Used to comment the config only. Optional
      - comment:
        public_key:
        preshared_key:
        allowed_ips: []
        # Endpoint address
        endpoint:
        persistent_keepalive:
```

The `wg-quick` method will setup the tunnel, peers and addresses. If
`systemd-networkd` is used, only a `.netdev` is created, and the user is
free to create their own `.network` to attach addresses on it. `wireguard-conf`
will just generate a configuration in `/etc/wireguard/` than can be used by
`wg`.


Example Playbook
----------------

```yaml

- hosts: wireguard_servers
  vars:
    wg-test0:
      method: "wg-quick"
      enable: True
      listen_port: 51820

      private_key: "0FZwbRzUF1ZG2i4hqhr1+oJJAQ3NTJDZDhpX3c1Qz1g="

      addresses:
        - "192.0.2.0/31"
        - "2001:db8::/127"
      dns:
        - "192.0.2.1"

      peers:
        - comment: "Some client"
          public_key: "yEvY7Jm8hgWLE64ocDMpwvcE3MH27xac6u55I2R2tik="
          allowed_ips:
            - "192.0.2.1"
            - "2001:db8::1/127"
          endpoint: "203.0.113.5"
          persistent_keepalive: 25

    wg-test1:
      method: "wireguard-conf"
      enable: False
      listen_port: 51821

      private_key: "0FZwbRzUF1ZG2i4hqhr1+oJJAQ3NTJDZDhpX3c1Qz1g="

      peers:
        - comment: "Another client"
          public_key: "WLE64ocDMpwvcyEvY7Jm8hgE3MH27xac6u55I2R2tik="
          allowed_ips:
            - "192.0.2.3"
          endpoint: "203.0.113.9"

  roles:
    - role: Anthony25.wireguard
```

And add the following line in `/etc/network/interfaces` to setup the second
interface.

```
auto wg-test1
iface wg-test1 inet static
        address 192.0.2.2
        netmask 255.255.255.254
        pre-up ip link add $IFACE type wireguard
        pre-up wg setconf $IFACE /etc/wireguard/$IFACE
        post-down ip link del $IFACE
iface wg-test1 inet6 static
        address 2001:db8::2
        netmask 127
```

Authors
-------

Anthony Ruhier (Anthony25)

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!
