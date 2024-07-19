# A Note on Exit Nodes

Currently it isn't clear how to set an exit node from a config file. The easiest means of achieving this is the following:

1. Make sure the "Locked" in the config is set to **false**
2. After starting the container, exec the `tailscale set --advertise-exit-node` command

