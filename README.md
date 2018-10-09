# Servers

## TODO

- Setup watchghost.


## rainette.afpy.org

Ces services sont arrêtés, on va pouvoir les virer, disons en décembre
2018, si personne ne pleure d'ici là :

- plone
- ldap
- la plupart des vieilles apps django et cubicweb


Ces services sont utiles :

- Nginx,
- un uwsgi pour le site web,
- la stack de mail (smtp, imap, antispam, antivirus) et de mailing list (mailman < 3.0).


### TODO

Il faut encore faire un dump static de http://paullaroid.pycon.fr/, ce
qui nous permettra d'arreter l'app paullaroid et un couchdb, mais il y
a un truc de navigation javascript que `wget` ne détecte pas et il ne
récupère donc pas toutes les pages.


## storage.afpy.org

### TODO

- Mises à jour.
- Documenter la sauvegarde.
