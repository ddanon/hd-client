# A Note on Exit Nodes

Currently it isn't clear how to set an exit node from a config file. The easiest means of achieving this is the following:

1. Make sure the "Locked" in the config is set to **false**

2. After starting the container, exec the `tailscale set --advertise-exit-node` command

# Headscale additional steps

Effiectively the steps are:

1. Find out the ID number of the node that should be an exit node

2. Enable routes on that node

One example of how a single pihole node showed up in headscale for me:

```bash
/ # headscale routes list

ID | Node   | Prefix    | Advertised | Enabled | Primary
1  | pihole | ::/0      | true       | false   | -
2  | pihole | 0.0.0.0/0 | true       | false   | -

/ # headscale routes enable -r 1

/ # headscale routes enable -r 2

/ # headscale routes list

ID | Node   | Prefix    | Advertised | Enabled | Primary
1  | pihole | ::/0      | true       | true    | -
2  | pihole | 0.0.0.0/0 | true       | true    | -

```

- [official docs](https://headscale.net/exit-node/#on-the-node)
