# Caddy Web Server on Docker for Raspberry Pi (arm7)

A [Docker](http://docker.com) image for [Caddy](http://caddyserver.com) to serve
http or https with auto-generated Let's Encrypt certs.

By default, this image is built with the following plugins:
-  [realip](http://caddyserver.com/docs/realip)
-  [filter](http://caddyserver.com/docs/filter)
-  [tls.dns.cloudflare](https://caddyserver.com/docs/tls.dns.cloudflare)

You can easily build your own image with your plugins of choice - refer to this
repo's [Dockerfile](./Dockerfile) for more info.

## Versions
  - Caddy Nightly
  - Compiled using Go v1.11.4
  - Built on Alpine Linux v3.7

## Getting Started

Simple usage:

```sh
$ docker run -d -p 2015:2015 detroitenglish/docker-caddy-rpi:latest
```

... and point your browser to `http://127.0.0.1:2015`

## Real World Example

```sh
$ docker run -d \
    --name caddy \
    --restart unless-stopped \
    -p 80:80 -p 443:443 \
    -v /var/www:/srv \
    -v $YOUR_CADDYFILEPATH:/etc/Caddyfile \
    -v /etc/ssl/caddy:/root/.caddy \
    -e CLOUDFLARE_EMAIL=$YOUR_CF_EMAIL \
    -e CLOUDFLARE_API_KEY=$YOUR_CF_API_KEY \
    -e SITE_DOMAIN=$YOUR_SITE_DOMAIN \
    -e BASICAUTH_USERNAME=$SECRET_USERNAME \
    -e BASICAUTH_PASSWORD=$SECRET_PASSWORD \
    detroitenglish/docker-caddy-rpi:latest \
    -email=$EMAIL_FOR_LETSENCRYPT_SIGNUP
```

...with example `Caddyfile` found in `$YOUR_CADDYFILEPATH`:
```
{$SITE_DOMAIN} {

  root /srv/{$SITE_DOMAIN}

  log / stdout "{combined}"
  errors stderr

  tls {
    dns cloudflare
  }

  realip cloudflare

  gzip

  filter rule {
    content_type text/html.*
    search_pattern foo
    replacement bar
  }

  basicauth /auth "{$BASICAUTH_USERNAME}" "{$BASICAUTH_PASSWORD}"

  rewrite /admin {
     if {$BASICAUTH_USER} match "."
     if {$BASICAUTH_PASSWORD} match "."
     to /auth{uri}
   }

  header / {
    -Server
    x-hello-from "caddy docker on raspberry pi"
  }

  proxy / 127.0.0.1:1234 {
    transparent
    without /auth
  }
}
```

Refer to the [docker run](https://docs.docker.com/engine/reference/commandline/run/) and
[Caddy web server](https://caddyserver.com/docs) documentation for help with above.

## Automagic https with Let's Encrypt

You must provide a valid domain FQDN pointing to your docker server, and provide
an email address parameter for LetsEncrypt signup and alerts.

Alternatively, you can provide your Cloudflare credentials, and validate via DNS records
by and using the following directive in your `Caddyfile`:

```
tls {
    dns cloudflare
}
```
