# A Note on Exit Nodes

Currently it isn't clear how to set an exit node from a config file. The easiest means of achieving this is the following:

1. Make sure the "Locked" in the config is set to **false**

2. After starting the container, exec the `tailscale set --advertise-exit-node` command

# Headscale additional steps

Copied description from Headscale docs below. Effiectively the steps are:

1. Find out the ID number of the node that should be an exit node

2. Enable routes on that node

```bash
$ # list nodes
$ headscale routes list
ID | Machine | Prefix    | Advertised | Enabled | Primary
1  |         | 0.0.0.0/0 | false      | false   | -
2  |         | ::/0      | false      | false   | -
3  | phobos  | 0.0.0.0/0 | true       | false   | -
4  | phobos  | ::/0      | true       | false   | -
$ # enable routes for phobos
$ headscale routes enable -r 3
$ headscale routes enable -r 4
$ # Check node list again. The routes are now enabled.
$ headscale routes list
ID | Machine | Prefix    | Advertised | Enabled | Primary
1  |         | 0.0.0.0/0 | false      | false   | -
2  |         | ::/0      | false      | false   | -
3  | phobos  | 0.0.0.0/0 | true       | true    | -
4  | phobos  | ::/0      | true       | true    | -
```

- [official docs](https://headscale.net/exit-node/#on-the-node)
