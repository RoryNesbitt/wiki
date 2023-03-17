---
title: Traefik Reverse Proxy
description:
published: 1
date: 2023-03-17T18:40:06.861Z
tags: 
editor: markdown
dateCreated: 2023-03-17T18:00:06.861Z
---

Traefik is the reverse proxy I use due to the simplicity of adding a new service when that service is running on the same docker network.
Firstly set up a custom docker network in unraid, I called mine rnd, with the command `docker network create rnd`.

> To set unraid to preserve custom networks:
> go to /Settings/DockerSettings
> disable docker
> set `Preserve user defined networks` to `true`
> reenable docker
{.is-info}

## tabs {.tabset}

### traefik.yml

This file is used to define the traefik config.
To reload this restart the traefik container.

### Explanation

Firstly we set the two entry points `web` and `websecure`. I have them set to forward web traffic to websecure in order to add TLS to every page by default.

This also includes a list of cloudflares public IP addresses for compatability. This isn't required if cloudflare isn't being used as the DNS host.

```yml
entryPoints:
  # Redirect everything to https
  web:
    address: :80
    forwardedHeaders:
      trustedIPs: &trustedIps
        # Start of Clouflare IPs
        - 173.245.48.0/20
        - 103.21.244.0/22
        - 103.22.200.0/22
        - 103.31.4.0/22
        - 141.101.64.0/18
        - 108.162.192.0/18
        - 190.93.240.0/20
        - 188.114.96.0/20
        - 197.234.240.0/22
        - 198.41.128.0/17
        - 162.158.0.0/15
        - 104.16.0.0/12
        - 172.64.0.0/13
        - 131.0.72.0/22
        - 2400:cb00::/32
        - 2606:4700::/32
        - 2803:f800::/32
        - 2405:b500::/32
        - 2405:8100::/32
        - 2a06:98c0::/29
        - 2c0f:f248::/32
        # End of Cloudlare IPs
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
```

Then for websecure instruct traefik to generate letsencrypt certs for each page and include any required middleware. These are defined in the `config.yml` file
```yml
  websecure:
    address: :443
    forwardedHeaders:
      # Reuse Cloudflare Trusted IPs
      trustedIPs: *trustedIps
    http:
      tls:
        # Generate a wildcard domain certificate
        certResolver: letsencrypt
        domains:
          - main: DOMAIN.COM
            sans:
              - '*.DOMAIN.COM'
      middlewares:
        - authelia@file # Remove this if authelia is not set up
        - securityHeaders@file
```

providers are used to define how traefik will create proxy entries.
file is used for manually creating entries, while docker is used to automatically create them for containers which are on the same docker network
```yml
providers:
  providersThrottleDuration: 2s
  file:
    filename: /etc/traefik/config.yml
    watch: true

  docker:
    watch: true
    network: rnd
    defaultRule: "Host(`{{ lower (trimPrefix `/` .Name )}}.DOMAIN.COM`)"
    swarmModeRefreshSeconds: 15s
    exposedByDefault: false # If set to true then traefik.enable=true will not be needed
```

Tell traefik to use letsencrypt for all 

```yml
certificatesResolvers:
  letsencrypt:
    acme:
      email: traefik.DOMAIN.COM
      storage: /etc/traefik/certs.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
```

Finally set some traefik settings

```yml
global:
  checkNewVersion: true
  sendAnonymousUsage: false

serversTransport:
  insecureSkipVerify: true

# Enable traefik ui
api:
  dashboard: true
  insecure: true

# Log level INFO|DEBUG|ERROR
log:
  level: INFO
```


### Complete file

```yml
entryPoints:
  # Redirect everything to https
  web:
    address: :80
    forwardedHeaders:
      trustedIPs: &trustedIps
        # Start of Clouflare IPs
        - 173.245.48.0/20
        - 103.21.244.0/22
        - 103.22.200.0/22
        - 103.31.4.0/22
        - 141.101.64.0/18
        - 108.162.192.0/18
        - 190.93.240.0/20
        - 188.114.96.0/20
        - 197.234.240.0/22
        - 198.41.128.0/17
        - 162.158.0.0/15
        - 104.16.0.0/12
        - 172.64.0.0/13
        - 131.0.72.0/22
        - 2400:cb00::/32
        - 2606:4700::/32
        - 2803:f800::/32
        - 2405:b500::/32
        - 2405:8100::/32
        - 2a06:98c0::/29
        - 2c0f:f248::/32
        # End of Cloudlare IPs
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https

  websecure:
    address: :443
    forwardedHeaders:
      # Reuse Cloudflare Trusted IPs
      trustedIPs: *trustedIps
    http:
      tls:
        # Generate a wildcard domain certificate
        certResolver: letsencrypt
        domains:
          - main: DOMAIN.COM
            sans:
              - '*.DOMAIN.COM'
      middlewares:
        - authelia@file # Remove this if authelia is not set up
        - securityHeaders@file

providers:
  providersThrottleDuration: 2s
  file:
    filename: /etc/traefik/config.yml
    watch: true

  docker:
    watch: true
    network: rnd
    defaultRule: "Host(`{{ lower (trimPrefix `/` .Name )}}.DOMAIN.COM`)"
    swarmModeRefreshSeconds: 15s
    exposedByDefault: false # If set to true then traefik.enable=true will not be needed

certificatesResolvers:
  letsencrypt:
    acme:
      email: traefik.DOMAIN.COM
      storage: /etc/traefik/certs.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"

global:
  checkNewVersion: true
  sendAnonymousUsage: false

serversTransport:
  insecureSkipVerify: true

# Enable traefik ui
api:
  dashboard: true
  insecure: true

# Log level INFO|DEBUG|ERROR
log:
  level: INFO
```


## tabs {.tabset}

### config.yml

This file is used to manually define an entry and to define middlewares.
This file will be automatically reloaded.

### Explanation 

#### Configuring Entries

First create a http router for each entry.
This is the rules for the URL such as adding a middleware to only one entry

> You can add multiple domains to one entry by adding them as a list under Host
> e.g. ``Host(`DOMAIN.COM`, `www.DOMAIN.COM`)``
{.is-info}

> This can be a completely different domain if you have multiple
{.is-success}
```yml
http:
  routers:
    service_one:
      rule: 'Host(`service1.DOMAIN.COM`)'
      service: serviceOne
    service_two:
      rule: 'Host(`service2.DOMAIN.COM`)'
      service: serviceTwo
      middlewares:
      - "ExtraMiddleware"
```

Then create the service which each are pointing to.
This is the settings for the local service

> Remember to set the URL to https if using port 443, just defining the port will not work
{.is-warning}

> port 80 and 443 are inferred and therefore do not need to be included, however doing so will not break anything
{.is-info}
```yml
  serviceOne:
    service:
      loadBalancer:
        servers:
          - url: https://IP:PORT
  serviceTwo:
    service:
      loadBalancer:
        servers:
          - url: https://IP
```

#### Configuring Middlewares

The security headers middleware adds custom html header tags. I don't pretend to understand this I just use it \:\)

```yml
  middlewares:
    # Security headers
    securityHeaders:
      headers:
        customResponseHeaders:
          X-Robots-Tag: "none,noarchive,nosnippet,notranslate,noimageindex"
          server: ""
          X-Forwarded-Proto: "https"
        sslProxyHeaders:
          X-Forwarded-Proto: https
        referrerPolicy: "strict-origin-when-cross-origin"
        hostsProxyHeaders:
          - "X-Forwarded-Host"
        customRequestHeaders:
          X-Forwarded-Proto: "https"
        contentTypeNosniff: true
        browserXssFilter: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsSeconds: 63072000
        stsPreload: true
```

The authelia middleware assumes that you have authelia set up on the same docker network. if not replace `http://authelia` with authelias IP address.
Authelia basic is included because it was in their docs, but I haven't made use of it

> When you replace DOMAIN.COM you can make this link to any subdomain or page and that is where authelia will redirect to.
{.is-info}

```yml
    authelia:
      forwardAuth:
        address: http://authelia:9091/api/verify?rd=https%3A%2F%2FDOMAIN.COM%2F
        trustForwardHeader: true
        authResponseHeaders:
          - "Remote-User"
          - "Remote-Groups"
          - "Remote-Email"
          - "Remote-Name"
          
    authelia-basic:
      forwardAuth:
        address: https://authelia:9091/api/verify?auth=basic
        trustForwardHeader: true
        authResponseHeaders:
          - "Remote-User"
          - "Remote-Groups"
          - "Remote-Email"
          - "Remote-Name"
```

### Complete file
```yml
http:
  routers:
    service_one:
      rule: 'Host(`service1.DOMAIN.COM`)'
      service: serviceOne
    service_two:
      rule: 'Host(`service2.DOMAIN.COM`)'
      service: serviceTwo
      middlewares:
      - "ExtraMiddleware"

  serviceOne:
    service:
      loadBalancer:
        servers:
          - url: https://IP:PORT
  serviceTwo:
    service:
      loadBalancer:
        servers:
          - url: https://IP

  middlewares:
    # Security headers
    securityHeaders:
      headers:
        customResponseHeaders:
          X-Robots-Tag: "none,noarchive,nosnippet,notranslate,noimageindex"
          server: ""
          X-Forwarded-Proto: "https"
        sslProxyHeaders:
          X-Forwarded-Proto: https
        referrerPolicy: "strict-origin-when-cross-origin"
        hostsProxyHeaders:
          - "X-Forwarded-Host"
        customRequestHeaders:
          X-Forwarded-Proto: "https"
        contentTypeNosniff: true
        browserXssFilter: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsSeconds: 63072000
        stsPreload: true
        
    authelia:
      forwardAuth:
        address: http://authelia:9091/api/verify?rd=https%3A%2F%2FDOMAIN.COM%2F
        trustForwardHeader: true
        authResponseHeaders:
          - "Remote-User"
          - "Remote-Groups"
          - "Remote-Email"
          - "Remote-Name"
          
    authelia-basic:
      forwardAuth:
        address: https://authelia:9091/api/verify?auth=basic
        trustForwardHeader: true
        authResponseHeaders:
          - "Remote-User"
          - "Remote-Groups"
          - "Remote-Email"
          - "Remote-Name"
```

## Adding a container

Once traefik has been configured it becomes extremely simple to add a new entry from the same server.
- Change that containers network to the custom docker network you created 
- Add a label `traefik.enable` and set it equal to true

> This can be further simplified by setting `exposedByDefault: true` and not needing the `traefik.enable` label
{.is-success}

### Changing the containers entry settings

The only time I have ever needed to manually change a setting is when I don't want the subdomain to match the container name, or the container has multiple ports and traefik has selected the wrong one. These can be set with:

- `traefik.http.services.SERVICE_NAME.loadbalancer.server.port` = `PORT`
- `traefik.http.routers.SERVICE_NAME.rule` = ``Host(`SUBDOMAIN.DOMAIN.COM`)``
