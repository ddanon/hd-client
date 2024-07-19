# Description

I made this repo so I can connect tailscale containers to my headscale server. Huge shoutout to [irbekrm](https://github.com/irbekrm) for the help!

The resulting compose file allows me to have one service per client, which makes use of MagicDNS to achieve the following:

Without this setup:

```
  _____________________________
 |                             |
 |     my-main-machine + TS    |
 |                             |
 | - - - - - - - - - - - - - - |
 | Jellyfin Container   :8096  |
 | - - - - - - - - - - - - - - |
 | Some Other Container :12345 |
 | - - - - - - - - - - - - - - |
 | Yet Another Container :80   |
 |_____________________________|

 Jellyfin connection = `http://my-main-machine:8096`
 Some Other connection = `http://my-main-machine:12345`
 Yet Another connection = `http://my-main-machine:80`
```

With this setup:

```
  _________________________________
 |                                 |
 |     my-main-machine             |
 |                                 |
 | - - - - - - - - - - - - - - - - |
 | Contailner + Jellyfin   + NGINX |
 | - - - - - - - - - - - - - - - - |
 | Contailner + :12345 srv + NGINX |
 | - - - - - - - - - - - - - - - - |
 | Contailner + :80 srv    + NGINX |
 |_________________________________|

 Jellyfin connection = `http://jellyfin.services.[my-domain].org`
 Some Other connection = `http://some-other-service.services.[my-domain].org`
 Yet Another connection = `http://yet-another-service.services.[my-domain].org`
```


While it seems like a small thing, here are the primary benefits I'm after:
- makes self hosted service access more natrual for less tech-literate users of my head(tail?)net
- makes use of MagicDNS so I don't have to manage IPs and ports manually
- with some tweaks, gives me a dashboard of _service health_ (not just system connection status) across my network
- allows for _very_ easy addition of services, regardless of what hardware or LAN they happen to be physically on

# Assumptions

- you want to create a services user to run these under
- you're using rootless podman(-compose) for both the headscale server and the tailscale container client (like I did)
- you already have a running headscale server, built from: _**docker.io/headscale/headscale:0.23.0-alpha12-debug**_
- you're using a newer version of podman -- I use podman & podman-compose from nixpkgs-stable (currently podman 5.1.2 and podman-compose 1.2.0)
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

5. [on the client] replace the ServerURL and AuthKey values in `cap-97.hujson`

```json
{
  ...
  "ServerURL": "https://[your domain here!!]",
  "AuthKey": "replace this string with the authkey from step 3!!",
  ...
}
```

6. `podman-compose up -d`

# Warnings

I tried porting this over to a raspberry pi 4 running the latest raspberry pi OS as of 20240718. The compose files herein **failed** with podman-compose v1.0.3.


# References 

As always, don't take my word for it. Read the docs if anything seems off!

- [tailscale container docs](https://tailscale.com/kb/1282/docker)
- [my issue on the tailscale repo, discussion that led to making this repo](https://github.com/tailscale/tailscale/issues/12831)
- [tailscale dockerhub page](https://hub.docker.com/r/tailscale/tailscale)
- [headscale - running on docker docs](https://github.com/juanfont/headscale/blob/main/docs/running-headscale-container.md)

