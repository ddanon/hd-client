Final instructions for anyone following behind me.

# Assumptions

- you want to create a services user to run these under
- you're using rootless podman(-compose) for both the headscale server and the tailscale container client (like I did)
- you already have a running headscale server, built from: _**docker.io/headscale/headscale:0.23.0-alpha12-debug**_

# Instructions

1. `git pull https://github.com/ddanon/hd-client.git`

2. [on the running server] create the user

```bash
# assuming your headscale container is called headscale-ct
podman exec headscale-ct headscale users create services
```

3. [on the running server] create the authkey

```bash
# assuming your headscale container is called headscale-ct
# change the expiration if you want the connection to last more than a day
# or omit it for a non-expiring key
podman exec headscale-ct headscale authkey create -u services --reusable --expiration 24h
```

4. [on the client] optional -- edit the compose.yml to add a custom service

```compose.yml
services:
  ## [contailner same as the posted file]

  ## Insert service here
  nginx:
    image: docker.io/nginx
    depends_on:
      - tailscale-nginx
    network_mode: service:tailscale-nginx
  
```

5. [on the client] replace the ServerURL and AuthKey values.

```json
{
  ...
  "ServerURL": "https://[your domain here!!]",
  "AuthKey": "replace this string with the authkey from step 3!!",
  ...
}
```

6. `podman-compose up -d`

As always, don't take my word for it. Read the docs if anything seems off!

- [headscale - running on docker docs](https://github.com/juanfont/headscale/blob/main/docs/running-headscale-container.md)
