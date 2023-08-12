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

    root /var/www/html/EPv2;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
    }

    location ~* \.txt$ {
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

#### nginx cloudflare realip
Cloudflare agi comme un proxy et attribue une IP à chaque visiteur, pour pouvoir recuperer l'IP reelle du visiteur il faut ajouter ce fichier de configuration dans **/etc/nginx/conf.d**

nginx-cloudflase-realip.conf

    set_real_ip_from 103.21.244.0/22;
    set_real_ip_from 103.22.200.0/22;
    set_real_ip_from 103.31.4.0/22;
    set_real_ip_from 104.16.0.0/12;
    set_real_ip_from 108.162.192.0/18;
    set_real_ip_from 131.0.72.0/22;
    set_real_ip_from 141.101.64.0/18;
    set_real_ip_from 162.158.0.0/15;
    set_real_ip_from 172.64.0.0/13;
    set_real_ip_from 173.245.48.0/20;
    set_real_ip_from 188.114.96.0/20;
    set_real_ip_from 190.93.240.0/20;
    set_real_ip_from 197.234.240.0/22;
    set_real_ip_from 198.41.128.0/17;
    set_real_ip_from 2400:cb00::/32;
    set_real_ip_from 2606:4700::/32;
    set_real_ip_from 2803:f800::/32;
    set_real_ip_from 2405:b500::/32;
    set_real_ip_from 2405:8100::/32;
    set_real_ip_from 2c0f:f248::/32;
    set_real_ip_from 2a06:98c0::/29;

    real_ip_header CF-Connecting-IP;
    #real_ip_header X-Forwarded-For;
