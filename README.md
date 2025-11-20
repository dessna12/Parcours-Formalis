# 🔮 PARCOURS -  Préparer son environnement de travail & Modélisation 

## 🎯 Objectif

Tu es chargé de **concevoir et déployer une plateforme d’apprentissage en ligne** dans un environnement sécurisé et industrialisé.

Ce projet te permettra de manipuler :
-  **Linux** : gestion des fichiers, permissions, processus et réseau  
-  **Docker** : orchestration d’un environnement applicatif complet  
-  **Modélisation** : conception d’une base de données complexe  
-  **Sécurité** : HTTPS, certificats SSL et reverse proxy  
-  **Automatisation** : tâches planifiées et scripts système  

---

## 🎬 Contexte

L’entreprise **Formalis** souhaite mettre en ligne une plateforme d’apprentissage moderne. Les apprenants doivent pouvoir suivre des **cours**, **remettre des travaux** et **recevoir des évaluations** des formateurs. 

Ce sont des cours en ligne pour lesquels des **forums/questions** liées au cours sont définies. La plateforme proposera également un **dashboard de statistiques** pour les apprenants leur permettant de suivre leur **évolution et leur activité**. 

A la fin de chaque formation ils pourront également évaluer cette formation par un **système d'avis** (notes + commentaires)


## 🧩 Partie 1 — Mise en place de l’environnement applicatif

L'objectif de cette partie est de déployer une application complète avec :
- Un **serveur Node.js** (API)
- Une **base MySQL**
- Un **reverse proxy NGINX**
- Un **certificat SSL Let's Encrypt**
- Un **script Bash** pour le renouvellement automatique du certificat

---

### 1. 🐈 Dépôt Git
- Crée un dépôt Git et configure un `.gitignore`.  
- Il aura une branche `dev`et une branche `main` qui sera la branche de production 

---

### 2. 🧱 Structure du projet
Organise le projet la manière suivante :

```
./formalis/
 ├── node-app/
 ├── nginx/
 ├── mysql/
 ├── scripts/
 └── docker-compose.yml
```

---

### 3. 🐋 Docker

Containeriser chaque composant (Node.js, MySQL, NGINX) et orchestrer le tout avec Docker Compose pour obtenir un environnement reproductible.

**🔧 Etapes:**

- **Images & Dockerfile**
  - Chaque service « applicatif » doit disposer d’une image construite (Node.js : image spécifique construite depuis un Dockerfile local).
  - Définir clairement le contexte de build et les dépendances au niveau du Dockerfile.
  - Documenter les variables d’environnement nécessaires à la construction et à l’exécution.

- **Docker Compose**
  - Orchestrer les services via Docker Compose : au minimum `node-app`, `db` (MySQL) et `nginx`.
  - Définir les dépendances entre services (order de démarrage / depends_on).
  - Exposer uniquement les ports nécessaires (éviter d’exposer la BDD sur l’hôte).
  - Monter des volumes pour la persistance des données MySQL.
  - Monter un volume pour partager les fichiers statiques entre NGINX et le projet si nécessaire.

- **Réseau & isolation**
  - Créer un réseau Docker dédié pour l’application, afin d’isoler la communication entre services.
  - S’assurer que les services communiquent par noms de service (DNS Docker), pas par localhost.

- **Volumes & persistance**
  - Configurer un volume persistant pour MySQL (données).
  - Prévoir un emplacement pour les logs applicatifs (volume) afin de faciliter la collecte.
  - **Les certificats SSL ne doivent pas être copiés dans le projet** ; Certbot tourne sur l’hôte et stocke les certificats dans `/etc/letsencrypt` (sur l’hôte). NGINX dans le conteneur doit lire ces certificats via un montage du répertoire `/etc/letsencrypt` de l’hôte vers le conteneur NGINX.

- **Sécurité & secrets**
  - Ne pas mettre de mots de passe en dur dans les fichiers committés.
  - Utiliser des variables d’environnement ou un mécanisme de secrets local (documenter où les credentials sont stockés et comment les injecter au runtime).

- **Healthchecks**
  - Prévoir des mécanismes de healthcheck pour les services critiques (ex. endpoint santé pour Node.js, connexion SQL pour MySQL).
  - Documenter comment vérifier l’état des services (commande Docker, logs).

- **Logs & debugging**
  - Expliquer comment consulter les logs des conteneurs et comment rediriger les logs applicatifs vers un volume.
  - Prévoir une commande ou procédure de nettoyage (stop, down, suppression des volumes si nécessaire) et expliquer les risques.

- **Initialisation de la base**
  - Prévoir une stratégie d’initialisation : script SQL d’import pour créer le schéma (MLD) et les données de test.
  - Documenter la manière d’exécuter ce script via Docker (par exemple via un conteneur d’init ou un mécanisme d’import au démarrage).

- **Commandes opérationnelles attendues**
  - Construire les images.
  - Lancer la stack en mode détaché.
  - Consulter les logs d’un service précis.
  - Exécuter une commande dans le conteneur Node pour tester la connexion à la BDD.
  - Arrêter la stack proprement.

- **Différence dev / prod**
  - Documenter les différences attendues entre un environnement de développement et un environnement de production (ex. volumes montés vs images immuables, debug activé, rebuild fréquent en dev).
  - Expliquer comment passer d’un environnement à l’autre via des variables d’environnement ou des fichiers de configuration.

---

### 4. 📱 Application Node.js
- Crée une petite API avec au moins une route de test (`/api/health`).  
- Connecte-la à la base MySQL du conteneur.  
- Vérifie la bonne communication entre les services.

---

### 5. 📊  Base de données
- Déploie MySQL via Docker Compose.  
- Crée la base `formalis_db`.  

---

### 6. 🔁 Reverse proxy NGINX
- Configure un conteneur NGINX pour :
  - Rediriger `/api/` vers le conteneur Node.js  
  - Servir la racine du site si vous avez un service de front(page statique ou placeholder)  
- Monte le répertoire `/etc/letsencrypt` (de l’hôte) dans le conteneur NGINX pour que celui-ci puisse lire les certificats.

---

### 7. 🔐 Certificat HTTPS
- Installe **Certbot** sur le système Linux (hors Docker).  
- Génére un certificat Let's Encrypt pour `formation.local`.  
- Les certificats doivent rester dans :

`/etc/letsencrypt/live/formation.local/`

- Configure NGINX (dans le conteneur) pour utiliser ces certificats montés depuis l’hôte.

👉🏻 Pour comprendre un peu mieux ce qu'il se passe : 

<details>
  <summary>HTTPS et certificats</summary>

    Pour sécuriser les échanges entre le navigateur et votre application, nous utilisons HTTPS. Cela nécessite un certificat SSL. Dans ce TP, nous utilisons un certificat auto-signé ou généré localement pour le domaine formation.local. Cela permet de chiffrer les communications même si le certificat n’est pas reconnu par le navigateur. 

</details>

<details>
  <summary>Certificat et nom de domaine</summary>

    Let’s Encrypt ne délivre des certificats que pour des noms de domaine publics. Comme nous travaillons sur un domaine local (formation.local), nous utilisons un certificat généré sur votre machine. Le navigateur peut afficher un avertissement de sécurité, ce qui est normal pour un certificat local.
</details>

<details>
  <summary>Montage du certificat dans NGINX</summary>

    Les fichiers du certificat ne sont pas stockés dans le projet, mais sur votre système Linux, dans /etc/letsencrypt/live/formation.local/. NGINX lit ce certificat depuis ce répertoire grâce à un montage de volume Docker, ce qui permet à vos conteneurs de fonctionner en HTTPS sans copier les fichiers dans le projet.
</details>

<details>
  <summary>Renouvellement automatique </summary>

    Même si le certificat local n’expire pas rapidement comme un certificat Let’s Encrypt réel, nous vous demandons de créer un script de renouvellement et une tâche cron. Cela permet de simuler la maintenance d’un certificat en production et de comprendre les bonnes pratiques DevOps.
</details>


#### Tester votre API en HTTPS

Pour tester vos routes backend via HTTPS avec formation.local, vous pouvez utiliser la commande 
```bash 
curl -k https://formation.local/api/health. 
```

L’option -k permet d’accepter un certificat auto-signé.



---

### 8. 🤖 Automatisation
- Crée un **script Bash** qui renouvelle le certificat (commande `certbot renew` ou équivalent).  
- Configure une **tâche cron** sur l’hôte pour exécuter ce script périodiquement (tous les 90 jours dans la nuit).  
- Redirige la sortie du script vers un fichier log dans `/var/log/`.

---

## 🧩 Partie 2 — Vérifications & validation

### 🔍 Points de contrôle

| Vérification | Méthode attendue | Résultat attendu |
|---------------|------------------|------------------|
| Conteneurs actifs | Commande Docker | Tous les services démarrés |
| Connexion Node ↔ MySQL | Requête API | Retour attendu |
| Reverse proxy NGINX | Accès via HTTPS | Requête redirigée vers l’API |
| Certificat SSL valide | Inspection système | Certificat Let's Encrypt fonctionnel |
| Cron opérationnel | Vérification des logs | Exécution mensuelle planifiée |

---

## ⚙️ Contraintes techniques

- Utilisation obligatoire de **Docker Compose**  
- Pas d’installation locale de Node.js ou MySQL  
- Le certificat SSL doit être géré via **Let's Encrypt** sur l’hôte  
- Les certificats doivent résider dans `/etc/letsencrypt` (hôte)  
- NGINX dans le conteneur doit lire les certificats via montage depuis l’hôte  
- NGINX doit servir le site exclusivement en **HTTPS**  
- Le script et la tâche cron doivent être vérifiables  

---


## 🧩 Partie 3 — Conception & spécifications

Tu dois produire la **conception fonctionnelle et technique** ainsi que la **modélisation de la base de données** de cette plateforme.

---

### ⚙️ Spécifications fonctionnelles attendues

Rédige un document listant les fonctionnalités principales du système :


- Gestion des utilisateurs (rôles : apprenant, formateur, administrateur)  
- Consultation des cours et des chapitres  
- Téléversement de ressources ou de devoirs  
- Évaluations et notation  
- Inscription aux cours  
- Gestion des sessions et des accès  

Chaque fonctionnalité doit préciser :
- L’acteur concerné  
- La description fonctionnelle  
- Le résultat attendu  

--- 

### 🧠 Modélisation des données

Conçois une base de données cohérente dont les données à modéliser sont les suivantes :

- **Utilisateurs** (étudiants, formateurs, administrateurs)
- **Cours** (titre, description, niveau, prix, statut, date de publication)
- **Chapitres** / Leçons liés à un cours
- **Inscriptions** (étudiant ↔ cours, avec date, état, progression)
- **Avis** (notation + commentaire)
- **Paiements** (un utilisateur peut acheter plusieurs cours, facture, moyen de paiement, statut)
- **Catégories** et tags de cours
- **Forum / Questions** liées à un cours (avec réponses)
- **Logs d’activité** (connexion, consultation de cours, progression, etc.)

Tu peux ajouter d’autres tables si nécessaire (catégories, messages, badges, etc.).

Attendus :

- Un MCD complet (Merise ou UML)
- Un MLD normalisé (3FN minimum)

Bonus : ajouter des contraintes d’intégrité

---

### 🧱 Spécifications techniques attendues

Complète avec un second document décrivant les choix techniques :
- Technologies utilisées (Node.js, MySQL, NGINX, Docker, Certbot)  
- Architecture logicielle et réseau  
- Variables d’environnement nécessaires  
- Ports exposés et communication entre conteneurs  
- Emplacement des fichiers de logs et des certificats  

---


#### 📦 Livrables attendus
- **MCD** (Modèle Conceptuel de Données)  
- **MLD** (Modèle Logique de Données)  
- **Documents de spécifications** (fonctionnelles + techniques)

---



## 📁 Livrables finaux

1. Documents de **spécifications fonctionnelles et techniques**  
2. **MCD** et **MLD** complets  
3. Arborescence du projet et configuration Docker  
4. Fichiers de configuration NGINX (sans certificats)  
5. Script Bash de renouvellement et crontab (exportée)  
6. Lien vers le dépôt Git (branches et commits visibles)  
7. Rapport de validation (captures, vérifications, tests)

---


### 💡 Bonus
- Ajout d’un conteneur **phpMyAdmin** ou **Adminer** pour tester la BDD (documenter les risques).  
- Supervision avec un script Bash de “health check” de l’API. 

---

**Fin du TP — Bon déploiement 🚀**

