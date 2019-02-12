# Letsencrypt role

This role uses the standalone mode of certbot if no webserver is
running (typically during the first installation), else uses the nginx
module.

Note that existing certificates are renewed (using the nginx module)
as a cron task/systemd timer.

It creates snippets in `/etc/nginx/snippets/letsencrypt-{{ fqdn }}.conf`.
