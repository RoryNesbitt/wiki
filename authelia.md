---
title: Authlia authentication with LDAP
description: 
published: 1
date: 2023-04-16T18:36:35.831Z
tags: 
editor: markdown
dateCreated: 2023-04-15T18:38:00.184Z
---

Authelia provides site wide login authentication with either a file of usernames or an LDAP server backend.

> For creating secrets you can use [allkeysgenerator](https://www.allkeysgenerator.com/Random/Security-Encryption-Key-Generator.aspx)
{.is-info}


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


```yml
password_policy:

  ## The standard policy allows you to tune individual settings manually.
  standard:
    enabled: false

    ## Require a minimum length for passwords.
    min_length: 8

    ## Require a maximum length for passwords.
    max_length: 0

    ## Require uppercase characters.
    require_uppercase: true

    ## Require lowercase characters.
    require_lowercase: true

    ## Require numeric characters.
    require_number: true

    ## Require special characters.
    require_special: true

  ## zxcvbn is a well known and used password strength algorithm. It does not have tunable settings.
  zxcvbn:
    enabled: true

    ## Configures the minimum score allowed.
    min_score: 3

access_control:

  ## Default policy can either be 'bypass', 'one_factor', 'two_factor' or 'deny'. It is the policy applied to any
  ## resource if there is no policy to be applied to the user.
  default_policy: one_factor
  rules:
    ## Rules applied to everyone
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

session:
  ## The name of the session cookie.
  name: authelia_session

  ## The domain to protect.
  ## Note: the authenticator must also be in that domain.
  ## If empty, the cookie is restricted to the subdomain of the issuer.
  domain: DOMAIN.COM

  ## Sets the Cookie SameSite value. Possible options are none, lax, or strict.
  ## Please read https://www.authelia.com/c/session#same_site
  same_site: lax

  ## The secret to encrypt the session data. This is only used with Redis / Redis Sentinel.
  ## Secret can also be set using a secret: https://www.authelia.com/c/secrets
  secret: "SECRET"

  ## The time before the cookie expires and the session is destroyed if remember me IS NOT selected.
  expiration: 1h

  ## The inactivity time before the session is reset. If expiration is set to 1h, and this is set to 5m, if the user
  ## does not select the remember me option their session will get destroyed after 1h, or after 5m since the last time
  ## Authelia detected user activity.
  inactivity: 5m

  ## The time before the cookie expires and the session is destroyed if remember me IS selected.
  ## Value of -1 disables remember me.
  remember_me_duration: 1M

  redis:
    host: redis
    port: 6379

    ## Username used for redis authentication. This is optional and a new feature in redis 6.0.
    # username: authelia

    ## Password can also be set using a secret: https://www.authelia.com/c/secrets
    password: PASSWORD

    ## This is the Redis DB Index https://redis.io/commands/select (sometimes referred to as database number, DB, etc).
    database_index: 0

    ## The maximum number of concurrent active connections to Redis.
    maximum_active_connections: 8

    ## The target number of idle connections to have open ready for work. Useful when opening connections is slow.
    minimum_idle_connections: 0

regulation:
  ## The number of failed login attempts before user is banned. Set it to 0 to disable regulation.
  max_retries: 3

  ## The time range during which the user can attempt login before being banned. The user is banned if the
  ## authentication failed 'max_retries' times in a 'find_time' seconds window. Find Time accepts duration notation.
  ## See: https://www.authelia.com/c/common#duration-notation-format
  find_time: 10m

  ## The length of time before a banned user can login again.
  ban_time: 12h

## The available providers are: `local`, `mysql`, `postgres`. You must use one and only one of these providers.
storage:
  ## The encryption key that is used to encrypt sensitive information in the database. Must be a string with a minimum
  ## length of 20. Please see the docs if you configure this with an undesirable key and need to change it, you MUST use
  ## the CLI to change this in the database if you want to change it from a previously configured value.
  encryption_key: "SECRET"
  mysql:
    host: mariadb
    port: 3306
    database: authelia
    username: authelia
    ## Password can also be set using a secret: https://www.authelia.com/c/secrets
    password: PASSWORD
    timeout: 5s

notifier:
  ## You can disable the notifier startup check by setting this to true.
  disable_startup_check: false
  smtp:
    ## The SMTP host to connect to.
    host: smtp-relay.sendinblue.com

    ## The port to connect to the SMTP host on.
    port: 587

    ## The connection timeout.
    timeout: 5s

    ## The username used for SMTP authentication.
    username: sendinblue@DOMAIN.COM

    ## The password used for SMTP authentication.
    ## Can also be set using a secret: https://www.authelia.com/c/secrets
    password: "API_KEY"

    ## The sender is used to is used for the MAIL FROM command and the FROM header.
    ## If this is not defined and the username is an email, we use the username as this value. This can either be just
    ## an email address or the RFC5322 'Name <email address>' format.
    sender: "'Account RND' <noreply@DOMAIN.COM>"

    ## HELO/EHLO Identifier. Some SMTP Servers may reject the default of localhost.
    identifier: localhost

    ## Subject configuration of the emails sent. {title} is replaced by the text from the notifier.
    subject: "[RND] {title}"

    ## This address is used during the startup check to verify the email configuration is correct.
    ## It's not important what it is except if your email server only allows local delivery.
    startup_check_address: test@authelia.com

    ## By default we require some form of TLS. This disables this check though is not advised.
    disable_require_tls: false

    ## Disables sending HTML formatted emails.
    disable_html_emails: false

    tls:
      ## Skip verifying the server certificate entirely. In preference to setting this we strongly recommend you add the
      ## certificate or the certificate of the authority signing the certificate to the certificates directory which is
      ## defined by the `certificates_directory` option at the top of the configuration.
      ## It's important to note the public key should be added to the directory, not the private key.
      ## This option is strongly discouraged but may be useful in some self-signed situations where validation is not
      ## important to the administrator.
      skip_verify: false

      ## Minimum TLS version for the connection.
      minimum_version: TLS1.2
...
```

### Complete file

```yml
theme: dark

jwt_secret: "SECRET"

## Default redirection URL
default_redirection_url: https://auth.DOMAIN.COM/

## Set the default 2FA method for new users and for when a user has a preferred method configured that has been
## disabled. This setting must be a method that is enabled.
## Options are totp, webauthn, mobile_push.
default_2fa_method: ""

server:

  ## The address to listen on.
  host: 0.0.0.0

  ## The port to listen on.
  port: 9091

  ## Set the single level path Authelia listens on.
  ## Must be alphanumeric chars and should not contain any slashes.
  path: ""

  ## Enables the pprof endpoint.
  enable_pprof: false

  ## Enables the expvars endpoint.
  enable_expvars: false

  ## Disables writing the health check vars to /app/.healthcheck.env which makes healthcheck.sh return exit code 0.
  ## This is disabled by default if either /app/.healthcheck.env or /app/healthcheck.sh do not exist.
  disable_healthcheck: false

  ## Authelia by default doesn't accept TLS communication on the server port. This section overrides this behaviour.
  tls:
    ## The path to the DER base64/PEM format private key.
    key: ""

    ## The path to the DER base64/PEM format public certificate.
    certificate: ""

    ## The list of certificates for client authentication.
    client_certificates: []

  headers:

    ## The CSP Template. Read the docs.
    csp_template: ""

log:
  ## Level of verbosity for logs: info, debug, trace.
  level: info

telemetry:

  metrics:
    ## Enable Metrics.
    enabled: false

    ## The address to listen on for metrics. This should be on a different port to the main server.port value.
    address: tcp://0.0.0.0:9959

## Parameters used for TOTP generation.
totp:
  ## Disable TOTP.
  disable: false

  ## The issuer name displayed in the Authenticator application of your choice.
  issuer: DOMAIN.COM

  ## The TOTP algorithm to use.
  ## It is CRITICAL you read the documentation before changing this option:
  ## https://www.authelia.com/c/totp#algorithm
  algorithm: sha1

  ## The number of digits a user has to input. Must either be 6 or 8.
  ## Changing this option only affects newly generated TOTP configurations.
  ## It is CRITICAL you read the documentation before changing this option:
  ## https://www.authelia.com/c/totp#digits
  digits: 6

  ## The period in seconds a one-time password is valid for.
  ## Changing this option only affects newly generated TOTP configurations.
  period: 30

  ## The skew controls number of one-time passwords either side of the current one that are valid.
  ## Warning: before changing skew read the docs link below.
  skew: 1
  ## See: https://www.authelia.com/c/totp#input-validation to read
  ## the documentation.

  ## The size of the generated shared secrets. Default is 32 and is sufficient in most use cases, minimum is 20.
  secret_size: 32

## Parameters used for WebAuthn.
webauthn:
  ## Disable Webauthn.
  disable: false

  ## Adjust the interaction timeout for Webauthn dialogues.
  timeout: 60s

  ## The display name the browser should show the user for when using Webauthn to login/register.
  display_name: Authelia

  ## Conveyance preference controls if we collect the attestation statement including the AAGUID from the device.
  ## Options are none, indirect, direct.
  attestation_conveyance_preference: indirect

  ## User verification controls if the user must make a gesture or action to confirm they are present.
  ## Options are required, preferred, discouraged.
  user_verification: preferred

## This is used to validate the servers time is accurate enough to validate TOTP.
ntp:
  ## NTP server address.
  address: "time.cloudflare.com:123"

  ## NTP version.
  version: 4

  ## Maximum allowed time offset between the host and the NTP server.
  max_desync: 3s

  ## Disables the NTP check on startup entirely. This means Authelia will not contact a remote service at all if you
  ## set this to true, and can operate in a truly offline mode.
  disable_startup_check: false

  ## The default of false will prevent startup only if we can contact the NTP server and the time is out of sync with
  ## the NTP server more than the configured max_desync. If you set this to true, an error will be logged but startup
  ## will continue regardless of results.
  disable_failure: false

## The available providers are: `file`, `ldap`. You must use only one of these providers.
authentication_backend:

  ## Password Reset Options.
  password_reset:
    ## Disable both the HTML element and the API for reset password functionality.
    disable: false

    ## External reset password url that redirects the user to an external reset portal. This disables the internal reset
    ## functionality.
    custom_url: ""

  ## The amount of time to wait before we refresh data from the authentication backend. Uses duration notation.
  refresh_interval: 5m

  ldap:
    implementation: custom

    ## The url to the ldap server. Format: <scheme>://<address>[:<port>].
    ## Scheme can be ldap or ldaps in the format (port optional).
    url: ldap://10.10.10.5

    ## The dial timeout for LDAP.
    timeout: 5m

    ## Use StartTLS with the LDAP connection.
    start_tls: false

    tls:

      ## Skip verifying the server certificate entirely. In preference to setting this we strongly recommend you add the
      ## certificate or the certificate of the authority signing the certificate to the certificates directory which is
      ## defined by the `certificates_directory` option at the top of the configuration.
      ## It's important to note the public key should be added to the directory, not the private key.
      ## This option is strongly discouraged but may be useful in some self-signed situations where validation is not
      ## important to the administrator.
      skip_verify: false

      ## Minimum TLS version for the connection.
      minimum_version: TLS1.2

    ## The distinguished name of the container searched for objects in the directory information tree.
    ## See also: additional_users_dn, additional_groups_dn.
    base_dn: dc=DOMAIN,dc=COM

    ## The attribute holding the username of the user. This attribute is used to populate the username in the session
    username_attribute: uid

    ## The additional_users_dn is prefixed to base_dn and delimited by a comma when searching for users.
    ## i.e. with this set to OU=Users and base_dn set to DC=a,DC=com; OU=Users,DC=a,DC=com is searched for users.
    additional_users_dn: cn=users,cn=accounts

    ## The users filter used in search queries to find the user profile based on input filled in login form.
    ## Various placeholders are available in the user filter which you can read about in the documentation which can
    ## be found at: https://www.authelia.com/c/ldap#users-filter-replacements
    ##
    ## Recommended settings are as follows:
    ## - Microsoft Active Directory: (&({username_attribute}={input})(objectCategory=person)(objectClass=user))
    ## - OpenLDAP:
    ##   - (&({username_attribute}={input})(objectClass=person))
    ##   - (&({username_attribute}={input})(objectClass=inetOrgPerson))
    ##
    ## To allow sign in both with username and email, one can use a filter like
    ## (&(|({username_attribute}={input})({mail_attribute}={input}))(objectClass=person))
    users_filter: (&({username_attribute}={input})(objectClass=person))

    ## The additional_groups_dn is prefixed to base_dn and delimited by a comma when searching for groups.
    ## i.e. with this set to OU=Groups and base_dn set to DC=a,DC=com; OU=Groups,DC=a,DC=com is searched for groups.
    additional_groups_dn: cn=users,cn=accounts

    ## The groups filter used in search queries to find the groups based on relevant authenticated user.
    ## Various placeholders are available in the groups filter which you can read about in the documentation which can
    ## be found at: https://www.authelia.com/c/ldap#groups-filter-replacements
    ##
    ## If your groups use the `groupOfUniqueNames` structure use this instead:
    ##    (&(uniqueMember={dn})(objectClass=groupOfUniqueNames))
    groups_filter: (&(member={dn})(objectClass=groupOfNames))

    ## The attribute holding the name of the group.
    group_name_attribute: cn

    ## The attribute holding the mail address of the user. If multiple email addresses are defined for a user, only the
    ## first one returned by the LDAP server is used.
    mail_attribute: mail

    ## The attribute holding the display name of the user. This will be used to greet an authenticated user.
    display_name_attribute: displayName

    ## Follow referrals returned by the server.
    ## This is especially useful for environments where read-only servers exist. Only implemented for write operations.
    permit_referrals: false

    ## The username and password of the admin user.
    user: uid=admin,cn=users,cn=accounts,dc=DOMAIN,dc=COM
    ## Password can also be set using a secret: https://www.authelia.com/c/secrets
    password: PASSWORD


password_policy:

  ## The standard policy allows you to tune individual settings manually.
  standard:
    enabled: false

    ## Require a minimum length for passwords.
    min_length: 8

    ## Require a maximum length for passwords.
    max_length: 0

    ## Require uppercase characters.
    require_uppercase: true

    ## Require lowercase characters.
    require_lowercase: true

    ## Require numeric characters.
    require_number: true

    ## Require special characters.
    require_special: true

  ## zxcvbn is a well known and used password strength algorithm. It does not have tunable settings.
  zxcvbn:
    enabled: true

    ## Configures the minimum score allowed.
    min_score: 3

access_control:

  ## Default policy can either be 'bypass', 'one_factor', 'two_factor' or 'deny'. It is the policy applied to any
  ## resource if there is no policy to be applied to the user.
  default_policy: one_factor
  rules:
    ## Rules applied to everyone
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

session:
  ## The name of the session cookie.
  name: authelia_session

  ## The domain to protect.
  ## Note: the authenticator must also be in that domain.
  ## If empty, the cookie is restricted to the subdomain of the issuer.
  domain: DOMAIN.COM

  ## Sets the Cookie SameSite value. Possible options are none, lax, or strict.
  ## Please read https://www.authelia.com/c/session#same_site
  same_site: lax

  ## The secret to encrypt the session data. This is only used with Redis / Redis Sentinel.
  ## Secret can also be set using a secret: https://www.authelia.com/c/secrets
  secret: "SECRET"

  ## The time before the cookie expires and the session is destroyed if remember me IS NOT selected.
  expiration: 1h

  ## The inactivity time before the session is reset. If expiration is set to 1h, and this is set to 5m, if the user
  ## does not select the remember me option their session will get destroyed after 1h, or after 5m since the last time
  ## Authelia detected user activity.
  inactivity: 5m

  ## The time before the cookie expires and the session is destroyed if remember me IS selected.
  ## Value of -1 disables remember me.
  remember_me_duration: 1M

  redis:
    host: redis
    port: 6379

    ## Username used for redis authentication. This is optional and a new feature in redis 6.0.
    # username: authelia

    ## Password can also be set using a secret: https://www.authelia.com/c/secrets
    password: PASSWORD

    ## This is the Redis DB Index https://redis.io/commands/select (sometimes referred to as database number, DB, etc).
    database_index: 0

    ## The maximum number of concurrent active connections to Redis.
    maximum_active_connections: 8

    ## The target number of idle connections to have open ready for work. Useful when opening connections is slow.
    minimum_idle_connections: 0

regulation:
  ## The number of failed login attempts before user is banned. Set it to 0 to disable regulation.
  max_retries: 3

  ## The time range during which the user can attempt login before being banned. The user is banned if the
  ## authentication failed 'max_retries' times in a 'find_time' seconds window. Find Time accepts duration notation.
  ## See: https://www.authelia.com/c/common#duration-notation-format
  find_time: 10m

  ## The length of time before a banned user can login again.
  ban_time: 12h

## The available providers are: `local`, `mysql`, `postgres`. You must use one and only one of these providers.
storage:
  ## The encryption key that is used to encrypt sensitive information in the database. Must be a string with a minimum
  ## length of 20. Please see the docs if you configure this with an undesirable key and need to change it, you MUST use
  ## the CLI to change this in the database if you want to change it from a previously configured value.
  encryption_key: "SECRET"
  mysql:
    host: mariadb
    port: 3306
    database: authelia
    username: authelia
    ## Password can also be set using a secret: https://www.authelia.com/c/secrets
    password: PASSWORD
    timeout: 5s

notifier:
  ## You can disable the notifier startup check by setting this to true.
  disable_startup_check: false
  smtp:
    ## The SMTP host to connect to.
    host: smtp-relay.sendinblue.com

    ## The port to connect to the SMTP host on.
    port: 587

    ## The connection timeout.
    timeout: 5s

    ## The username used for SMTP authentication.
    username: sendinblue@DOMAIN.COM

    ## The password used for SMTP authentication.
    ## Can also be set using a secret: https://www.authelia.com/c/secrets
    password: "API_KEY"

    ## The sender is used to is used for the MAIL FROM command and the FROM header.
    ## If this is not defined and the username is an email, we use the username as this value. This can either be just
    ## an email address or the RFC5322 'Name <email address>' format.
    sender: "'Account RND' <noreply@DOMAIN.COM>"

    ## HELO/EHLO Identifier. Some SMTP Servers may reject the default of localhost.
    identifier: localhost

    ## Subject configuration of the emails sent. {title} is replaced by the text from the notifier.
    subject: "[RND] {title}"

    ## This address is used during the startup check to verify the email configuration is correct.
    ## It's not important what it is except if your email server only allows local delivery.
    startup_check_address: test@authelia.com

    ## By default we require some form of TLS. This disables this check though is not advised.
    disable_require_tls: false

    ## Disables sending HTML formatted emails.
    disable_html_emails: false

    tls:
      ## Skip verifying the server certificate entirely. In preference to setting this we strongly recommend you add the
      ## certificate or the certificate of the authority signing the certificate to the certificates directory which is
      ## defined by the `certificates_directory` option at the top of the configuration.
      ## It's important to note the public key should be added to the directory, not the private key.
      ## This option is strongly discouraged but may be useful in some self-signed situations where validation is not
      ## important to the administrator.
      skip_verify: false

      ## Minimum TLS version for the connection.
      minimum_version: TLS1.2
...
```

## new part

