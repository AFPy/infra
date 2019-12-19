# Survol des playbooks Ansible

On découpe nos *playbooks* Ansible par rôles :

- `site.yml`: Inclu tous les autres, pratique pour tout exécuter.
- `pycon.yml`: Pour les pycon.fr
- `passbolt.yml`: Pour passbolt.
- `backup.yml`: Configure rsnapshot pour sauvegarder nos serveurs.
- ...

En partant de là, on peut utiliser les commandes suivantes:

Après avoir cloné ce repo, installé Ansible dans un venv, installez
les roles nécessaires via :

- ansible-galaxy install -r requirements.yml

Puis pour jouer les *playbooks* :

- Pour tout relancer : `ansible-playbook site.yml`
- Pour configurer les PyCons : `ansible-playbook pycons.yml`
- Pour configurer Passbolt : `ansible-playbook passbolt.yml`
  (attention voir [#15](https://github.com/laxathom/ansible-role-passbolt/issues/15)).


## Faire, ne pas faire

Faire : Configurer les machines : apt install, fichiers de
configuration, utilisateurs, éventuellement un premier `git clone`
pour que ça marche si c'est un site statique.

Ne pas faire : Deployer. En dehors de l'éventuel premier git clone,
c'est le rôle de la CI (Github Actions, ...), pas de nos playbooks.


# Servers

## TODO

- [ ] Github Actions sur Alain (déployer au push).
- [ ] Migrer afpyro.afpy.org
- [ ] Setup watchghost


## deb.afpy.org

La seule machine déployée via Ansible.

fingerprint: `SHA256:xVC4sYYdmDSbJP6JWZUxApzHdbAj1p3uZlOEIksXrMA`.


## rainette.afpy.org

Liste des jails toujours utiles :

- web: héberge https://afpy.org (https://github.com/afpy/site/)
- dns
- smtp:
  - smtpd (/usr/local/etc/mail/smtpd.conf)
  - dovecot (comptes: /usr/local/etc/mail/tables/passwd)
  - spamd
- pyconfr
  - Un static dump de 2013: `/home/pyconfr/static_dump/2013/`.
  - Un static dump de 2014: `/home/pyconfr/static_dump/2014/`.
  - Un static dump de 2015: `/home/pyconfr/static_dump/2015/`.
  - Les Pelican de 2016, 2017, 2018.
  - Le flask-freeze de 2019.
- mailman
  - Le sitepass est disponnible dans passbolt.
- http
  - C'est le nginx qui dispatch aux autres jails.
  - 2019-07-04: `rm sites-enabled/{plone,hg,paullaroid,membres,supervision}.afpy.org`
- alain
  - Bot IRC sur #afpy sur freenode.


Jails en fin de vie :

- afpyro
  - https://afpyro.afpy.org/
  - https://github.com/AFPy/siteafpyro
  - Il faudrait poser une 301 vers ... afpy.org ?


## storage.afpy.org

Accessible via https://dl.afpy.org

On y stocke :

- videospyconfr2016
- videospyconfr2015
- http
- backupdebian
- ns1


# Ansible

On utilies ces rôles Ansible :

## julienpalard.nginx

Voir la [doc](https://github.com/JulienPalard/ansible-role-nginx).


## common

*common* est un rôle "de base" permettant d'avoir une conf "normale"
sur toutes nos machines (emacs et vim installés, nos authorized-keys,
pas de mlocate, hostname propre, firewall, ce genre de broutilles).


# Backups

Julien Palard a un rsnapshot (en 2019 en tout cas) de storage.afpy.org
sur son NAS perso, avec :

```
backup  root@storage.afpy.org:/usr/jails/http/usr/local/www/nginx-dist/dl/      storage.afpy.org/

backup  root@rainette.afpy.org:/etc/    rainette.afpy.org

backup  root@rainette.afpy.org:/usr/jails/web/usr/local/etc/    rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/web/root/     rainette.afpy.org

backup  root@rainette.afpy.org:/usr/jails/dns/usr/local/etc/    rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/supervision/usr/local/etc/    rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/static/usr/local/etc/ rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/ldap/usr/local/etc/   rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/smtp/usr/local/etc/   rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/pyconfr/usr/local/etc/        rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/plone/usr/local/etc/  rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/couchdb/usr/local/etc/        rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/photomaton/usr/local/etc/     rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/membres/usr/local/etc/        rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/mailman/usr/local/etc/        rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/http/usr/local/etc/   rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/alain/usr/local/etc/  rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/afpyro/usr/local/etc/ rainette.afpy.org

backup  root@rainette.afpy.org:/usr/jails/http/usr/local/www/   rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/mailman/usr/local/www/        rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/pyconfr/usr/local/www/        rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/static/usr/local/www/ rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/supervision/usr/local/www/    rainette.afpy.org
backup  root@rainette.afpy.org:/usr/jails/web/usr/local/www/    rainette.afpy.org
```


## Passbolt

See [passbolt backup documentation](https://help.passbolt.com/hosting/backup).

On a un CRON qui lance un `mysqldump` vers `/srv/backups/passbolt.sql`
sur le serveur du passbolt, qui dont pourrait se faire sauvegarder par rsnapshot.
