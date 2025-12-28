<img width="935" height="989" alt="image" src="https://github.com/user-attachments/assets/649eb7b5-0f7d-40d2-bf33-eb3eaaa6fce0" />
<img width="902" height="928" alt="image" src="https://github.com/user-attachments/assets/31815093-eb6b-4f85-bcfc-cd1e458d7ada" />
<img width="848" height="193" alt="image" src="https://github.com/user-attachments/assets/d2e1b056-d5d7-4014-8991-f1e2d3eaaff9" />
<img width="1912" height="727" alt="image" src="https://github.com/user-attachments/assets/6ec5c148-b633-49cd-94a9-a43d9013d668" />

<img width="921" height="712" alt="image" src="https://github.com/user-attachments/assets/98d77441-75c9-495f-acf8-7fff001c1182" />
<img width="914" height="818" alt="image" src="https://github.com/user-attachments/assets/9a47a407-1c0a-45ca-8912-ce182252059f" />
<img width="710" height="577" alt="image" src="https://github.com/user-attachments/assets/e5781645-46e2-477f-ad5c-7feded5c8a08" />
<img width="913" height="584" alt="image" src="https://github.com/user-attachments/assets/3a3bab03-62b1-48a1-ad1d-158da660f32a" />
<img width="876" height="633" alt="image" src="https://github.com/user-attachments/assets/6d2feaaf-3866-4e1b-a885-99b341a559c0" />
<img width="966" height="619" alt="image" src="https://github.com/user-attachments/assets/dcbe3261-0d9d-418a-9faa-e5af68ae21d9" />
<img width="751" height="592" alt="image" src="https://github.com/user-attachments/assets/4ceb2442-4847-4730-8bd5-62b48d2997d1" />
<img width="715" height="578" alt="image" src="https://github.com/user-attachments/assets/9eb1260c-c966-4091-be85-e461865aa527" />


TP-26 — Architecture microservices avec Docker Compose
🎯 Objectif

Mettre en place une architecture microservices composée de :

1 base de données MySQL partagée

1 service pricing-service

3 instances du book-service

Vérification de la cohérence des données, du verrouillage DB, de la résilience et de la persistance

🧱 Architecture du stack
Service	Port externe	Port interne
pricing-service	8082	8082
book-service-1	8081	8081
book-service-2	8083	8081
book-service-3	8084	8081
mysql	—	3306

Chaque instance book-service est indépendante mais partage la même base MySQL.

🚀 Étape 7 — Démarrage du stack
Commande
docker compose up -d --build

Résultat

Tous les services sont construits

Les conteneurs démarrent

Les healthchecks sont exécutés automatiquement

❤️ Vérification de l’état de santé (Healthchecks)
pricing-service
curl http://localhost:8082/actuator/health


Résultat

status: UP

book-service-1
curl http://localhost:8081/actuator/health


Résultat

status: UP

book-service-2
curl http://localhost:8083/actuator/health


Résultat

status: UP

book-service-3
curl http://localhost:8084/actuator/health


Résultat

status: UP

🔁 Vérification multi-instances
Commandes
curl http://localhost:8081/api/debug/instance
curl http://localhost:8083/api/debug/instance
curl http://localhost:8084/api/debug/instance

Résultat observé
instance=91e9bd2b3015 internalPort=8081
instance=2c7d09ee7083 internalPort=8081
instance=04a9127604a8 internalPort=8081


✔️ Chaque instance possède un hostname différent

🧪 Étape 8 — Scénarios de validation
8.1 Données partagées (MySQL commun)
Création d’un livre (instance 1)
Invoke-RestMethod -Method POST http://localhost:8081/api/books `
  -ContentType "application/json" `
  -Body '{"title":"Dune","author":"Herbert","stock":3}'


Résultat

id title author  stock
1  Dune  Herbert 3

Lecture depuis les autres instances
Invoke-RestMethod http://localhost:8083/api/books
Invoke-RestMethod http://localhost:8084/api/books


Résultat

id title author  stock
1  Dune  Herbert 3


✔️ Même base de données partagée

8.2 Stock cohérent (verrou DB)
Emprunts concurrents
Invoke-RestMethod -Method POST http://localhost:8081/api/books/1/borrow
Invoke-RestMethod -Method POST http://localhost:8083/api/books/1/borrow
Invoke-RestMethod -Method POST http://localhost:8084/api/books/1/borrow
Invoke-RestMethod -Method POST http://localhost:8083/api/books/1/borrow

Résultat attendu

3 emprunts réussis

4ᵉ emprunt rejeté

HTTP 409 Conflict
{"error":"Plus d’exemplaires"}


✔️ Verrouillage DB fonctionnel

8.3 Résilience — pricing-service arrêté
Arrêt du service pricing
docker compose stop pricing-service

Emprunt avec pricing down
Invoke-RestMethod -Method POST http://localhost:8081/api/books/2/borrow


Résultat

id title      stockLeft price
2  Foundation 1         0.0


✔️ Fallback activé (price=0.0)

Redémarrage pricing-service
docker compose start pricing-service

8.4 Persistance MySQL (volume Docker)
Arrêt complet du stack
docker compose down

Redémarrage
docker compose up -d

Vérification des données
Invoke-RestMethod http://localhost:8081/api/books


Résultat

id title      author   stock
1  Dune       Herbert 0
2  Foundation Asimov  1


✔️ Les données sont conservées après redémarrage

✅ Conclusion

Architecture multi-instances opérationnelle

Données partagées et cohérentes

Gestion correcte de la concurrence

Résilience aux pannes de service

Persistance garantie par volume Docker

🎉 TP validé avec succès
