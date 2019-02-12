# Servers

## TODO

- [ ] HTTPS sur paullaroid.pycon.fr, 2010.pycon.fr, 2011.pycon.fr
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
- smtp la stack de mail (smtp, imap, antispam, antivirus) et de mailing list (mailman < 3.0).
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

En parlant de backups, Julien Palard a un rsnapshot (en 2019) de
storage.afpy.org sur son NAS.


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
