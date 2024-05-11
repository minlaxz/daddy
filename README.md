# daddy


Caddy in docker and using Cloudflare and Route53.

It's usually very difficult for me to name something.

`daddy` supports two kinds of reverse-proxying docker containers over TLS.

1. [Using Cloudflare](https://github.com/minlaxz/daddy/main/README.md#using-cloudflare) _Please refer to [this](https://github.com/caddy-dns/cloudflare) for more information._
2. [Using Let's Encrypt](https://github.com/minlaxz/daddy/main/README.md#using-lets-encrypt)

### Using Let's Encrypt
Let's create a docker network first. For example, bridge/ external docker network.
```
docker network create caddy-net

# Example daddy docker-compose file.
# docker-compose.yml
services:
    caddy:
      image: ghcr.io/minlaxz/daddy:main
      ports:
        - 80:80
        - 443:443
      environment:
        - CADDY_INGRESS_NETWORKS=caddy-net
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock
        - caddy_data:/data
      restart: unless-stopped

networks:
    default:
      name: caddy-net
      external: true

volumes:
    caddy_data: {}
```
And spin it up, `docker compose up -d`

Application container
```sh
services:
    whoami:
      image: traefik/whoami
      labels:
        caddy: whoami.metaforce.minlaxz.lol
        caddy.reverse_proxy: "{{upstreams 80}}"

networks:
  default:
    name: caddy-net
    external: true
```

### Using Cloudflare
Application container
```sh
services:
    portr-tunnel:
      image: amalshaji/portr-tunnel:0.0.15-beta
      command: ["start"]
      env_file: .env
      restart: unless-stopped
      ports:
        - "2222:2222"
      labels:
        caddy_1: "*.$DOMAIN"
        caddy_1.reverse_proxy: "{{upstreams http 8001}}"
        caddy_1.tls.dns: "cloudflare $CLOUDFLARE_API_TOKEN"
        caddy_1.encode: gzip
```


No need to mention, we also need to set a wildcard DNS record in our NS.
```
A | *.example.com | YOUR_EC2_PUBLIC_IP_OR_WHATEVER
```
