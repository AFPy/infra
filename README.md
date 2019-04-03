# Servers

Dependencies:
 - tschifftner.exim4_sendonly from ansible galaxy
 - https://github.com/laxathom/ansible-role-passbolt

## TODO

- [ ] Setup watchghost
- [ ] Rédiger le ansible pour afpy.org, en profiter pour mettre en place du continuous delivery.
- [ ] Documenter la sauvegarde.
- [ ] Sauvegarder puis supprimer la jail supervision
- [ ] Sauvegarder puis supprimer la jail static
- [ ] Sauvegarder puis supprimer la jail ldap
- [ ] Vérifier les versions des Django des pycon 2012, 2013, 2014, 2015
- [ ] Déployer les django des pycon 2012, 2013, 2014, 2015 via Ansible
- [ ] Sauvegarder puis supprimer la jail plone
- [ ] Sauvegarder puis supprimer la jail couchdb
- [ ] Sauvegarder puis supprimer la jail photomaton
- [ ] Sauvegarder puis supprimer la jail membres
- [ ] Rediriger le sous domaine afpyro.afpy.org vers autre chose.
- [ ] Sauvegarder puis supprimer la jail afpyro


## rainette.afpy.org

Liste des jails :

- web: héberge https://afpy.org (https://github.com/afpy/site/)
- dns
- supervision: Un icinga2, Julien, Voileux, et jpcw ont un compte.
   - abandonné depuis 2017
   - shooté le 2018-12-21.
- static
  - Héberge des fichiers sur afpy.org qui ne sont plus utilisés depuis le passage a flask.
- ldap (Arrêté le 21 décembre 2018)
- smtp:
  - smtpd (/usr/local/etc/mail/smtpd.conf)
  - dovecot (comptes: /usr/local/etc/mail/tables/passwd)
  - spamd
- pyconfr
  - Le cubicweb (https://www.pycon.fr/cw/) avec les pycon de 2009, 2010, 2011.
  - Les Django de 2012 2013 2014 2015.
  - Les Pelican de 2016 2017 2018.
- plone (Arrêté le 20 décembre 2018)
- couchdb
  - Utilisé par l'ancien photomaton, migré en statique, arrété le 2018-12-21.
- photomaton
  - Ancien photomaton en Pyramid, migré en statique, arrêté le 2018-12-21.
- membres:
  - Ancienne gestion des membres (https://github.com/AFPy/AfpyMembers)
  - Arrêté le 2018-12-21
- mailman
  - Le sitepass est disponnible dans passbolt.
- http C'est le nginx qui dispatch aux autres jails.
- alain
  - Bot IRC sur #afpy sur freenode.
- afpyro
  - https://afpyro.afpy.org/
  - https://github.com/AFPy/siteafpyro
  - Il faudrait poser une 301 vers ... afpy.org ?


## storage.afpy.org

On y stocke :

- videospyconfr2016
- videospyconfr2015
- http
- backupdebian
- ns1


## 163.172.45.2

Est une machine de test de Julien, elle héberge paullaroid.pycon.fr,
pycon2010.pycon.fr, pycon2011.pycon.fr déployés via ansible.


# Ansible

On utilies ces rôles Ansible :


## gallery

C'est le rôle pour installer https://github.com/AFPy/pycon-fr-gallery
sur http://paullaroid.pycon.fr/.

Une démo est actuellement sur une machine de test de Julien:

    curl --resolve paullaroid.pycon.fr:80:163.172.45.2 http://paullaroid.pycon.fr


## letsencrypt

*letsencrypt* est un rôle pour gérer un certificat HTTPS et son stub
nginx pour un domaine. Il s'utilise typiquement comme dépendance d'un
autre rôle, voir le `meta/main.yml` du rôle `gallery` par exemple.


## common

*common* est un rôle "de base" permettant d'avoir une conf "normale"
sur toutes nos machines (emacs et vim installés, nos authorized-keys,
pas de mlocate, ce genre de broutilles)


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

We're having a cron dumping mysql to /srv/backups/passbolt.sql on the
passbolt host so we can rsnapshot it (we rely on rsnapshot to keep historic versions).
