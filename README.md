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

## TODO

### Mailman 3

Mailman 3 est installé sur https://mm3.afpy.org, Julien a un compte
super user, vous pouvez en demander un aussi. le mailman peut envoyer
des mails via exim4, mais pour le moment aucune mailing list.


## Faire, ne pas faire

Faire : Configurer les machines : apt install, fichiers de
configuration, utilisateurs, éventuellement un premier `git clone`
pour que ça marche si c'est un site statique.

Ne pas faire : Deployer. En dehors de l'éventuel premier git clone,
c'est le rôle de la CI (Github Actions, ...), pas de nos playbooks.


# Servers

## TODO

- [ ] Github Actions sur Alain (déployer au push).
- [ ] Setup watchghost
- [ ] C'est quoi pycon.afpy.org ?
- [ ] C'est quoi video.pycon.fr ? (IN A 91.121.116.118)


## deb.afpy.org

La seule machine déployée via Ansible.

fingerprint: `SHA256:xVC4sYYdmDSbJP6JWZUxApzHdbAj1p3uZlOEIksXrMA`.


## rainette.afpy.org

Liste des jails toujours utiles :

- web: stoppée, sauvegardée, à supprimer.
- dns: Doit être démarrée avant mailman
- smtp:
  - smtpd (/usr/local/etc/mail/smtpd.conf)
  - dovecot (comptes: /usr/local/etc/mail/tables/passwd)
  - spamd
- mailman: Le sitepass est disponnible dans passbolt.
- http: toujours utile pour https://lists.afpy.org


## dl.afpy.org

C'est un directory listing nginx, accessible via https://dl.afpy.org.

Il héberge aussi https://videos-2015.pycon.fr/ (qui depuis 2021 redirige
vers dl.afpy.org/pycon-fr-15, c'est mieux rangé).


# Ansible

On utilies ces rôles Ansible :

## julienpalard.nginx

Voir la [doc](https://github.com/JulienPalard/ansible-role-nginx).


## common

*common* est un rôle "de base" permettant d'avoir une conf "normale"
sur toutes nos machines (emacs et vim installés, nos authorized-keys,
pas de mlocate, hostname propre, firewall, ce genre de broutilles).


# Backups

Julien Palard a un rsnapshot (vérifié en 2021) sur son NAS perso, avec :

```
backup  deb.afpy.org:/srv/      deb.afpy.org/
backup  deb.afpy.org:/home/     deb.afpy.org/
backup  deb.afpy.org:/etc/      deb.afpy.org/
backup  deb.afpy.org:/srv/      deb.afpy.org/
backup  deb.afpy.org:/home/     deb.afpy.org/
backup  deb.afpy.org:/etc/      deb.afpy.org/
backup  deb.afpy.org:/var/discourse/shared/standalone/backups/  deb.afpy.org/
backup  storage.afpy.org:/var/www/ storage.afpy.org/
```


## Passbolt

See [passbolt backup documentation](https://help.passbolt.com/hosting/backup).

On a un CRON qui lance un `mysqldump` vers `/srv/backups/passbolt.sql`
sur le serveur du passbolt, qui dont pourrait se faire sauvegarder par rsnapshot.


## BBB

On a installé le BBB simplement, sur bbb.afpy.org, une machine dédiée :

```
root@bbb:~# wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -v xenial-22 -s bbb.afpy.org -e julien@palard.fr -w -g
```
