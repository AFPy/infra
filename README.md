# Survol des playbooks Ansible

On découpe nos *playbooks* Ansible par rôles :

- `site.yml`: Inclu tous les autres, pratique pour tout exécuter.
- `pycon.yml`: Pour les pycon.fr
- `backup.yml`: Configure rsnapshot pour sauvegarder nos serveurs.
- ...

En partant de là, on peut utiliser les commandes suivantes:

Après avoir cloné ce repo, installé Ansible dans un venv, installez
les roles nécessaires via :

- ansible-galaxy install -r requirements.yml

Puis pour jouer les *playbooks* :

- Pour tout relancer : `ansible-playbook site.yml`
- Pour configurer les PyCons : `ansible-playbook pycons.yml`

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
- mailman: Le sitepass est disponnible dans [pass](https://github.com/AFPy/pass/).
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


## BBB

On utilise une Start-2-M-SATA chez online.net qui propose directement d'installer BBB.

Attention a retenir son mot de passe user temporaire à l'installation
même si vous avez une clef SSH: vous en aurez besoin pour `passwd` la
première fois.

J'y ai appliqué un poil de ssh-hardening :

    AuthenticationMethods publickey
    LogLevel VERBOSE

Et quand même un peu de confort (enfin c'est surtout que je n'aime pas
les mots de passe ni sudo) :

    PermitRootLogin prohibit-password


Ensuite j'ai [rsync les enregistrements depuis le bbb
précédent](https://docs.bigbluebutton.org/2.2/customize.html#transfer-published-recordings-from-another-server).

Puis j'ai sauvegardé/restauré la DB de greenlight :

    # Sur l'ancienne machine :
    docker exec greenlight_db_1 /usr/bin/pg_dumpall -U postgres -f /var/lib/postgresql/data/dump.sql

    # Sur la nouvelle machine :
    # Copier la sauvegarde sur le nouveau serveur :
    cd ~root/greenlight
    rsync bbb.afpy.org:/root/greenlight/db/production/dump.sql ./

    docker-compose down
    rm -fr db
    # Configurer le même mot de passe dans .env et docker-compose.yml que l'ancienne machine
    # En profiter pour vérifier le SAFE_HOSTS dans le .env.
    docker-compose up -d
    # Attendre un peu avec un top sous les yeux que ça se termine vraiment
    docker exec greenlight_db_1 /usr/local/bin/psql -U postgres -c "DROP DATABASE greenlight_production;"
    mv dump.sql db/production/
    docker exec greenlight_db_1 /usr/local/bin/psql -U postgres -f /var/lib/postgresql/data/dump.sql
    rm db/production/dump.sql
    docker-compose down
    docker-compose up -d  # Il va s'occuper de la migration
    docker-compose logs -f # pour voir si tout va bien

`rsync` des certificats TLS aussi :

    rsync -vah bbb.afpy.org:/etc/letsencrypt/ /etc/letsencrypt/

Ça a pris un petit :

    sed s/sd-106563.dedibox.fr/bbb.afpy.org/ /etc/nginx/sites-available/bigbluebutton

Il faut attendre un moment avec un `top` qui tourne, ruby a tout plein
de truc a faire avant de démarrer.
