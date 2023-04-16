---
title: Authlia authentication with LDAP
description: 
published: 1
date: 2023-04-16T19:21:36.987Z
tags: 
editor: markdown
dateCreated: 2023-04-15T18:38:00.184Z
---

Authelia provides site wide login authentication with either a file of usernames or an LDAP server backend.

> For creating secrets you can use [allkeysgenerator](https://www.allkeysgenerator.com/Random/Security-Encryption-Key-Generator.aspx)
{.is-info}

## Prerequisits
- A reverse proxy such as [Traefik](/traefik)
- Redis
- (Optional) An LDAP server such as freeIPA or Active Directory
- (Optional|Recommended) A mysql database such as mariaDB
- (Optional) an SMTP provider. I recommend [Send In Blue](/sendinblue)

## tabs {.tabset}

### configuration.yml

This file is used to define the authelia config.
To reload this restart the authelia container.

> A default will be created when authelia is first run. It is advisable to edit that rather than replacing it completely.
{.is-warning}

### Explanation

Make it dark because you're not a savage
```yml
theme: dark
```

This secret is used for validating users identity
```yml
jwt_secret: "SECRET"
```

Usually authelia will redirect back to whatever you were trying to access, this link is used if you go directly to the authelia url
```yml
default_redirection_url: https://DOMAIN.COM/
```

I don't know why this is empty but it works, I'll update this when I know more
```yml
default_2fa_method: ""
```

Settings for the server itself, most of this just lets it be happy about any incoming traffic
```yml
server:
  host: 0.0.0.0
  port: 9091
  path: ""
  enable_pprof: false
  enable_expvars: false
  disable_healthcheck: false
  tls:
    key: ""
    certificate: ""
    client_certificates: []
  headers:
    csp_template: ""
```

Set the log level and disable telemetry
```yml
log:
  level: info

telemetry:
  metrics:
    enabled: false
    address: tcp://0.0.0.0:9959
```

totp and webauth are two of the TFA options, the other is mobile_push
```yml
totp:
  disable: false
  issuer: DOMAIN.COM
  algorithm: sha1
  digits: 6
  period: 30
  skew: 1
  secret_size: 32

webauthn:
  disable: false
  timeout: 60s
  display_name: Authelia
  attestation_conveyance_preference: indirect
  user_verification: preferred

```

Get the time from cloudflares ntp address
```yml
ntp:
  address: "time.cloudflare.com:123"#
  version: 4
  max_desync: 3s
  disable_startup_check: false
  disable_failure: false
```

The authentication backend is where you get the use accounts from. Set the basics here
```yml
authentication_backend:
  password_reset:
    disable: false
    custom_url: ""
  refresh_interval: 5m
```

I use LDAP with freeIPA so that's whats here. The other option is a yaml file, it's simpler but it's annnoying to add to
```yml
  ldap:
    implementation: custom
    url: ldap://IP:PORT
    timeout: 5m
    start_tls: false
    tls:
      skip_verify: false
      minimum_version: TLS1.2
```

This part tells authelia how to understand LDAP, if using freeIPA then these settings should be the same for you, just change DOMAIN COM and PASSWORD
```yml
    base_dn: dc=DOMAIN,dc=COM
    username_attribute: uid
    additional_users_dn: cn=users,cn=accounts
    users_filter: (&({username_attribute}={input})(objectClass=person))
    additional_groups_dn: cn=users,cn=accounts
    groups_filter: (&(member={dn})(objectClass=groupOfNames))
    group_name_attribute: cn
    mail_attribute: mail
    display_name_attribute: displayName
    permit_referrals: false
    user: uid=admin,cn=users,cn=accounts,dc=DOMAIN,dc=COM
    password: PASSWORD
```

Set which password policy to use. Standard is basically custom, zxcbn is a better option
```yml
password_policy:
  standard:
    enabled: false
    min_length: 8
    max_length: 0
    require_uppercase: true
    require_lowercase: true
    require_number: true
    require_special: true

  zxcvbn:
    enabled: true
    min_score: 3
```

Access control is where you set the different URLs to different user groups, I'll break it down in a different section
```yml
access_control:
  default_policy: one_factor
  rules:
    - domain: # Public accessable
        - "public.DOMAIN.COM"
        - "public2.DOMAIN.COM"
      policy: bypass
    - domain: # api access
        - "*.DOMAIN.COM"
      resources:
        - "^/api([/?].*)?$"
      policy: bypass
    - domain: # Admin group
        - "admin.DOMAIN.COM"
        - "admin2.DOMAIN.COM"
      subject:
        - "group:admin"
      policy: two_factor
```

Settings for the session cookie. This mostly effects times you don't click remember me
```yml
session:
  name: authelia_session
  domain: DOMAIN.COM
  same_site: lax
  secret: "SECRET"
  expiration: 1h
  inactivity: 5m
  remember_me_duration: 1M
```

Redis is an in RAM database, I think. You might need to change the index if you use it for other things
```yml
  redis:
    host: redis
    port: 6379
    password: PASSWORD
    database_index: 0
    maximum_active_connections: 8
    minimum_idle_connections: 0
```

When to ban a user and how long it should last
```yml
regulation:
  max_retries: 3
  find_time: 10m
  ban_time: 12h
```

I use mariaDB for my storage but you can use a local file or postgres if you want. Just set the database up with full access to the authelia user
```yml
storage:
  encryption_key: "SECRET"
  mysql:
    host: mariadb
    port: 3306
    database: authelia
    username: authelia
    password: PASSWORD
    timeout: 5s
```

This is for if you to send out emails for things like password resets. Send in blue lets you send something like 300 emails per day for free so that's what I use. Most of the settings you will get from there, they are pretty easy to use.
The account name can be anything you like and be from any email you like, I just like to use the noreply one because I drop all incoming emails to it.

If you don't want that just comment this out
```yml
notifier:
  disable_startup_check: false
  smtp:
    host: smtp-relay.sendinblue.com
    port: 587
    timeout: 5s
    username: USER_EMAIL
    password: "API_KEY"
    sender: "'Account RND' <noreply@DOMAIN.COM>"
    identifier: localhost
    subject: "[SITENAME] {title}"
    startup_check_address: test@authelia.com
    disable_require_tls: false
    disable_html_emails: false
    tls:
      skip_verify: false
      minimum_version: TLS1.2
```
### Access Control

The default policy will act on all URLs not specified. Other than adding this everything here goes top down, so it will act on the first match in the list. You can set this to block in order to never let through anything you don't specify, but that seems tedious to me.
```yml
access_control:
  default_policy: one_factor
```

Setting URLs to bypass will let you access them without authentification
```yml
  rules:
    - domain: # Public accessable
        - "public.DOMAIN.COM"
        - "public2.DOMAIN.COM"
      policy: bypass
```

This entry adds a bypass on all subdomains but only for api access. The resource is the limitation here that lets that work. If you don't have this entry then some webapps might break 
```yml
    - domain: # api access
        - "*.DOMAIN.COM"
      resources:
        - "^/api([/?].*)?$"
      policy: bypass
```

The next entry is for giving access to only people part of a group. For this example I have made it available to the non-existant group X and also to anyone with admin rights. I have also set these to require TFA
```yml
    - domain: # group X
        - "x.DOMAIN.COM"
        - "x2.DOMAIN.COM"
      subject:
        - "group:X"
        - "group:admin"
      policy: two_factor
```

### Complete file

```yml
theme: dark

jwt_secret: "SECRET"

default_redirection_url: https://DOMAIN.COM/

default_2fa_method: ""

server:
  host: 0.0.0.0
  port: 9091
  path: ""
  enable_pprof: false
  enable_expvars: false
  disable_healthcheck: false
  tls:
    key: ""
    certificate: ""
    client_certificates: []
  headers:
    csp_template: ""

log:
  level: info

telemetry:
  metrics:
    enabled: false
    address: tcp://0.0.0.0:9959

totp:
  disable: false
  issuer: DOMAIN.COM
  algorithm: sha1
  digits: 6
  period: 30
  skew: 1
  secret_size: 32

webauthn:
  disable: false
  timeout: 60s
  display_name: Authelia
  attestation_conveyance_preference: indirect
  user_verification: preferred

ntp:
  address: "time.cloudflare.com:123"#
  version: 4
  max_desync: 3s
  disable_startup_check: false
  disable_failure: false

authentication_backend:
  password_reset:
    disable: false
    custom_url: ""
  refresh_interval: 5m

  ldap:
    implementation: custom
    url: ldap://IP:PORT
    timeout: 5m
    start_tls: false
    tls:
      skip_verify: false
      minimum_version: TLS1.2

    base_dn: dc=DOMAIN,dc=COM
    username_attribute: uid
    additional_users_dn: cn=users,cn=accounts
    users_filter: (&({username_attribute}={input})(objectClass=person))
    additional_groups_dn: cn=users,cn=accounts
    groups_filter: (&(member={dn})(objectClass=groupOfNames))
    group_name_attribute: cn
    mail_attribute: mail
    display_name_attribute: displayName
    permit_referrals: false
    user: uid=admin,cn=users,cn=accounts,dc=DOMAIN,dc=COM
    password: PASSWORD

password_policy:
  standard:
    enabled: false
    min_length: 8
    max_length: 0
    require_uppercase: true
    require_lowercase: true
    require_number: true
    require_special: true

  zxcvbn:
    enabled: true
    min_score: 3

access_control:

  default_policy: one_factor
  rules:
    - domain: # Public accessable
        - "public.DOMAIN.COM"
        - "public2.DOMAIN.COM"
      policy: bypass
    - domain: # api access
        - "*.DOMAIN.COM"
      resources:
        - "^/api([/?].*)?$"
      policy: bypass
    - domain: # group X
        - "x.DOMAIN.COM"
        - "x2.DOMAIN.COM"
      subject:
        - "group:X"
        - "group:admin"
      policy: two_factor
      policy: two_factor

session:
  name: authelia_session
  domain: DOMAIN.COM
  same_site: lax
  secret: "SECRET"
  expiration: 1h
  inactivity: 5m
  remember_me_duration: 1M

  redis:
    host: redis
    port: 6379
    password: PASSWORD
    database_index: 0
    maximum_active_connections: 8
    minimum_idle_connections: 0

regulation:
  max_retries: 3
  find_time: 10m
  ban_time: 12h

storage:
  encryption_key: "SECRET"
  mysql:
    host: mariadb
    port: 3306
    database: authelia
    username: authelia
    password: PASSWORD
    timeout: 5s

notifier:
  disable_startup_check: false
  smtp:
    host: smtp-relay.sendinblue.com
    port: 587
    timeout: 5s
    username: USER_EMAIL
    password: "API_KEY"
    sender: "'Account RND' <noreply@DOMAIN.COM>"
    identifier: localhost
    subject: "[SITENAME] {title}"
    startup_check_address: test@authelia.com
    disable_require_tls: false
    disable_html_emails: false
    tls:
      skip_verify: false
      minimum_version: TLS1.2
...
```

## new part

