## Redirection du nom de domaine `ankorez.fr` vers GitHub Pages

1. **Crée un fichier CNAME** :
Sur ton dépôt GitHub, crée un fichier nommé "CNAME" à la racine du dépôt. Ce fichier doit contenir uniquement ton nom de domaine, sans aucun préfixe "http://" ou "https://". Par exemple, dans ton cas, le contenu du fichier CNAME sera simplement : `ankorez.fr`.

2. **Configure le DNS de ton nom de domaine** :
 Accède à l'interface de gestion de ton domaine chez ton fournisseur d'hébergement de noms de domaine (où tu as acheté ankorez.fr). Trouve la section de gestion DNS ou des enregistrements DNS.

3. **Ajoute un enregistrement CNAME** :
   Crée un nouvel enregistrement CNAME et configure-le comme suit :
   - Nom : laisse ce champ vide ou utilise "@" pour indiquer le domaine racine (ankorez.fr).
   - Cible ou cible du lien : entre ton nom d'utilisateur GitHub suivi de ".github.io". Par exemple, si ton nom d'utilisateur GitHub est "ankorez", la cible sera : `ankorez.github.io`.

4. **Enregistre les modifications** :
   Une fois que tu as configuré l'enregistrement CNAME, enregistre les modifications dans l'interface de gestion DNS.

Note que les modifications du DNS peuvent prendre un certain temps à se propager à travers Internet (habituellement quelques heures, mais cela peut prendre jusqu'à 48 heures). Après cela, lorsque tu saisiras "ankorez.fr" dans un navigateur, tu seras redirigé vers ta page GitHub.

Assure-toi également que ton dépôt GitHub contient une page index.html ou un fichier par défaut, sinon GitHub affichera une erreur 404. Tu peux utiliser le fichier README.md comme page d'accueil si tu n'as pas de fichier HTML spécifique.

En cas de problème, vérifie que tu as correctement configuré le fichier CNAME dans ton dépôt GitHub et que l'enregistrement CNAME est correctement défini dans ton DNS.
