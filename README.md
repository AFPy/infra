# Survol des playbooks Ansible

On découpe nos *playbooks* Ansible par rôles :

- `pycon.yml`: Pour les pycon.fr
- `backup.yml`: Configure rsnapshot pour sauvegarder nos serveurs.
- ...

En partant de là, on peut utiliser les commandes suivantes:

Après avoir cloné ce repo, installé Ansible dans un venv, installez
les roles nécessaires via :

- ansible-galaxy install julienpalard.nginx

Récupérez le secret dans [afpy/pass](https://github.com/AFPy/pass/):

    git clone https://github.com/AFPy/pass/
    PASSWORD_STORE_DIR=pass/infra pass Ansible-Vault > ~/.ansible-afpy-vault

Puis pour jouer les *playbooks* :

- Pour tout relancer : `ansible-parallel *.yml`
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


## roles/nginx

Ce rôle configure un nginx avec Letsencrypt en DNS-01 via l'API Gandi (nos domaines étant chez Gandi).

L'avantage du DNS-01 c'est qu'on peut configurer un nouveau serveur **avant** que le DNS ne pointe sur lui.


## julienpalard.nginx

Ce rôle configure un nginx avec Letsencrypt en HTTP-01, on l'utilise
assez peu maintenant, on l'utilise là où on ne peut pas faire de
DNS-01 (pour `fr.pycon.org` par exemple).

Voir la [doc](https://github.com/JulienPalard/ansible-role-nginx).


## common

*common* est un rôle "de base" permettant d'avoir une conf "normale"
sur toutes nos machines (emacs et vim installés, nos authorized-keys,
pas de mlocate, hostname propre, firewall, ce genre de broutilles).


# Sauvegardes

Gandi nous fournit un serveur de sauvegardes, cf. l'inventaire et `backup.yml`.

Julien Palard a aussi un rsnapshot (vérifié en 2022) sur son NAS perso, au cas où.


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


### BBB password reset

Pour accepter le password reset, BBB doit avoir :

    ALLOW_MAIL_NOTIFICATIONS=true

dans /root/greenlight/.env

(Pour relire le `.env`: `cd /root/greenlight; docker-compose down && docker-compose up -d`)

Pour vérifier la conf :

    docker run --rm --env-file .env bigbluebutton/greenlight:v2 bundle exec rake conf:check

Il y a des chances que ça ne passe pas, il faut laisser les mails
sortir de leur conteneur Docker (par défaut il utilise sendmail DANS
le conteneur).

Il faut configurer le `.env` tel que:

    SMTP_SERVER=172.17.0.1
    SMTP_PORT=25
    SMTP_DOMAIN=greenlight.afpy.org
    SMTP_SENDER=bbb@afpy.org

Puis vérifier qu'exim et le firewall (attention c'est peut-être `ufw`)
les acceptent.


### Configuration TURN/STUN

L'installation de BBB n'étant pas gérée par Ansible, pour le moment la
conf TURN/STUN est faite à la main, c'est la seule chose à faire, elle
ressemble à :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
            ">

    <bean id="stun0" class="org.bigbluebutton.web.services.turn.StunServer">
        <constructor-arg index="0" value="stun:turn.afpy.org"/>
    </bean>

    <bean id="turn0" class="org.bigbluebutton.web.services.turn.TurnServer">
        <constructor-arg index="0" value="[redacte]"/>
        <constructor-arg index="1" value="turns:turn.afpy.org:443?transport=tcp"/>
        <constructor-arg index="2" value="86400"/>
    </bean>

    <bean id="stunTurnService" class="org.bigbluebutton.web.services.turn.StunTurnService">
        <property name="stunServers">
            <set>
                <ref bean="stun0" />
            </set>
        </property>
        <property name="turnServers">
            <set>
              <ref bean="turn0" />
            </set>
        </property>
        <property name="remoteIceCandidates">
            <set>
            </set>
        </property>
    </bean>
</beans>
```

dans `/usr/share/bbb-web/WEB-INF/classes/spring/turn-stun-servers.xml`.
