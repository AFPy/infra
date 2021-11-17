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


## Faire, ne pas faire

Faire : Configurer les machines : apt install, fichiers de
configuration, utilisateurs, éventuellement un premier `git clone`
pour que ça marche si c'est un site statique.

Ne pas faire : Deployer. En dehors de l'éventuel premier git clone,
c'est le rôle de la CI (Github Actions, ...), pas de nos playbooks.


# Servers

La distinction services/serveurs :

- Un serveur contient un chiffre dans son hostname : deb2.afpy.org,
  bbb2.afpy.org, …
- Un service ne contient pas de chiffre dans son hostname :
  discuss.afpy.org, bbb.afpy.org, www.afpy.org, …


## deb2.afpy.org

♥ Machine sponsorisée par Gandi ♥

C'est un VPS `V-R4 2 CPUs · 4 GB RAM`.

Elle héberge les services suivants :

- https://www.afpy.org ([source](https://github.com/AFPy/site))
- https://discuss.afpy.org une instance Discourse.
- [https://*.pycon.fr/*](https://pycon.fr/) (que des sites statiuques)
- https://afpyro.afpy.org ([source](https://github.com/AFPy/siteafpyro))
- Alain le bot IRC du canal #afpy ([source](https://github.com/AFPy/alain))
- La gate [IRC](https://afpy.org/irc)—[Discord](https://afpy.org/discord)
- https://dl.afpy.org: un *directory listing* nginx des vidéos de nos conférences.
- https://logs.afpy.org: Les logs du salon IRC #afpy ([source](https://github.com/AFPy/AfpyLogs/))
- https://pydocteur.afpy.org: Un bot utilisé dans le repo de la traduction ([source](https://github.com/AFPy/PyDocTeur))


## bbb2.afpy.org

♥ Machine sponsorisée par Gandi ♥

C'est un VPS `V-R8 4 CPUs · 8 GB RAM`.


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

Hébergé sur bbb2.afpy.org chez Gandi.

J'y ai appliqué un poil de ssh-hardening :

    AuthenticationMethods publickey
    LogLevel VERBOSE

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
