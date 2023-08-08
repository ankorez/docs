# How to make a simple website with the domain ankorez.fr

### Prérequis: 

 1. Une machine debian hebergée sur Internet
 2. Une clé SSH pour se connecter sur la machine debian
 3. Un nom de domaine
 4. Un compte sur cloudlfare
 5. Installer les paquets php et nginx

#### Créer une instance sur Scaleway
Sélectionner une instance (elle est souvent en rupture de stock car c'est la moins chère).
**STARDUST1**
Il faut avoir uploader sa clé SSH sur son compte Scaleway, ainsi elle sera ajoutée automatiquement à toute nouvelle instance créée
#### Se connecter à l'instance pour installer les paquets

    ssh root@ipdeinstance
    sudo apt update
    sudo apt install php-fpm nginx
#### Configurer son website
Par defaut la page nginx s'affiche lorsqu'on se rend sur notre siteweb.
Les repertoires **sites-available** et **sites-enabled** contiennent ce qu'on appelle des **vhost**
1.  **sites-available** (Sites disponibles) : Le répertoire "sites-available" contient les fichiers de configuration des sites web disponibles sur le serveur. Chaque fichier de configuration correspond généralement à un site spécifique. Ces fichiers contiennent les directives de configuration pour définir les paramètres du site, tels que le nom de domaine, les chemins d'accès aux fichiers, les options de sécurité, les règles de réécriture, etc.

Lorsqu'un nouvel hébergement de site web doit être ajouté ou configuré sur le serveur, l'administrateur système ou le développeur place généralement un fichier de configuration dans le répertoire "sites-available".

2.  **sites-enabled** (Sites activés) : Le répertoire "sites-enabled" contient des liens symboliques (raccourcis) vers les fichiers de configuration des sites qui doivent être activés et pris en compte par le serveur web. Contrairement à "sites-available", qui contient tous les fichiers de configuration disponibles, "sites-enabled" contient uniquement les configurations actives que le serveur doit traiter.

Les liens symboliques permettent d'activer ou de désactiver facilement un site sans avoir à déplacer ou supprimer physiquement le fichier de configuration. Pour activer un site, un lien symbolique est créé dans le répertoire "sites-enabled" qui pointe vers le fichier de configuration approprié dans "sites-available". De même, pour désactiver un site, le lien symbolique est simplement supprimé.

Voici la structure de mon **vhost** situé dans /etc/nginx/sites-available/ankorez.fr

    server {
        listen 80;
        server_name ankorez.fr www.ankorez.fr;
    
        root /var/www/html/monsite;
        index index.php index.html index.htm;
    
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }
    
        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php7.4-fpm.sock; # Assurez-vous que le chemin vers le socket PHP-FPM est correct pour votre configuration.
        }
    
        location ~ /\.ht {
            deny all;
        }
    }


Je sauvegarde le fichier **vhost** sous le nom **ankorez.fr**
Puis je créer un lien symbolique pour l'activer

    sudo ln -s /etc/nginx/sites-available/ankorez.fr /etc/nginx/sites-enabled/

Si je veux le desactiver je dois taper cette commande

    sudo unlink /etc/nginx/sites-enabled/ankorez.fr
Il faut bien penser à relancer nginx

    sudo service nginx reload
Pour afficher la liste des liens symboliques

    ls -l /etc/nginx/sites-enabled/
#### Se connecter sur bookmyname.com et acheter son nom de domaine

 1. Chercher si le nom de domaine ankorez.fr est disponible
 2. Acheter notre nom de domaine
 3. Gerer > 
