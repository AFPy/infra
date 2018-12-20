# Servers

## Ansible

On utilies ces rôles Ansible :

### gallery

C'est le rôle pour installer https://github.com/AFPy/pycon-fr-gallery
sur http://paullaroid.pycon.fr/.

Une démo est actuellement sur une machine de test de Julien:

    curl --resolve paullaroid.pycon.fr:80:163.172.45.2 http://paullaroid.pycon.fr

### letsencrypt

*letsencrypt* est un rôle pour gérer un certificat HTTPS et son stub
nginx pour un domaine. Il s'utilise typiquement comme dépendance d'un
autre rôle, voir le `meta/main.yml` du rôle `gallery` par exemple.

### common

*common* est un rôle "de base" permettant d'avoir une conf "normale"
sur toutes nos machines (emacs et vim installés, nos authorized-keys,
pas de mlocate, ce genre de broutilles)

## TODO

- Setup watchghost.


## rainette.afpy.org

Ces services sont arrêtés, on va pouvoir les virer, disons en décembre
2018, si personne ne pleure d'ici là :

- plone (arrêté le 20 décembre 2018)
- ldap
- la plupart des vieilles apps django et cubicweb


Ces services sont utiles :

- Nginx,
- un uwsgi pour le site web,
- la stack de mail (smtp, imap, antispam, antivirus) et de mailing list (mailman < 3.0).


### TODO

http://paullaroid.pycon.fr/ à été dump dans https://github.com/AFPy/pycon-fr-gallery
Il reste a le mettre en ligne (voir la démo ansible) et a shooter le couchdb.


## storage.afpy.org

### TODO

- Mises à jour.
- Documenter la sauvegarde.
