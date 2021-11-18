# Nginx with Letsencrypt

This role sets up nginx with letsencrypt (using DNS-01 with Gandi API) .


## Role Variables

The mandatory variables are:

- `admin_email`: For letsencrypt.
- `gandi_api_key` ([see doc](https://github.com/obynio/certbot-plugin-gandi/)).
- `nginx_certificates`: A list of domain to put in this certificate.
- `nginx_domain`: Used for file names, certificate name, and default server_name if no nginx_conf is given.
- `nginx_conf`: The nginx config.

Optional variables are:

- `nginx_owner`: If a unix user has to be created for this project.
- `nginx_path`: To create a directory owned by `nginx_owner`.
- `ssl_ciphers`: [Specifies the enabled ciphers](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_ciphers).
- `ssl_protocols`: [Enables the specified protocols.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols)
- `ssl_prefer_server_ciphers`: [Specifies that server ciphers should be preferred over client ciphers when using the SSLv3 and TLS protocols.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_prefer_server_ciphers)
- `ssl_session_cache`: [Sets the types and sizes of caches that store session parameters.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache).
- `HSTS_header`: HTTP header to inject.


### Author Information

Julien Palard — https://mdk.fr
