# DoodleStudent – DevOps Project

**Membres de l'équipe :** MEZIANE Zakariae - ALIOUA Nordine - NOUMDEM Karl

## Presentation
DoodleStudent est une application micro-services (Quarkus + Angular) modernisée avec une approche DevOps complète. Ce projet démontre la mise en œuvre de l'automatisation, de la résilience et de la gestion de configuration industrielle.

## Architecture
L'écosystème repose sur 5 piliers interconnectés :
- Frontend : Angular servie par un Reverse Proxy Nginx.
- Backend : API REST Quarkus (Java).
- Base de données : MySQL 8.0.
- Services tiers : Etherpad (édition) et SMTP (e-mails).

## Lancer le projet

### Avec Docker Compose (Rapide)
```bash
docker-compose up -d
```
*Lance instantanément l'intégralité de la stack et les réseaux isolés.*

### Avec Ansible (Déploiement Automatisé)
```bash
ansible-playbook -i infra/ansible/inventory.ini infra/ansible/playbook.yml
```
*Automatise le provisioning, la génération sécurisée du .env et le lancement de l'infrastructure.*

---

## 1. Conteneurisation & Orchestration

### Dockerisation Multi-Stage
Nous avons optimisé les images pour la production :
- Backend (api/Dockerfile) : Compilation via Maven 3.9 suivie d'un run sur JRE 17. Point clé : Installation de iproute2 pour permettre les tests de latence.
- Frontend (front/Dockerfile) : Build Angular suivi d'un service par Nginx Alpine. Configuration dynamique via templates.

### Orchestration avec Docker Compose
Le fichier docker-compose.yaml centralise la gestion :
- Réseaux : Isolation via app-network.
- Persistance : Volumes Docker pour MySQL.
- Dépendances : Utilisation de depends_on pour l'ordre de démarrage.

---

## 2. Gestion de Configuration

### Fichier .env
Toutes les variables sensibles sont externalisées pour éviter le hardcoding :
- DB_PASSWORD, DB_ROOT_PASSWORD
- BACKEND_URL, INTERNAL_PAD_URL

### Automatisation avec Ansible
Ansible a été choisi comme outil d'Infrastructure as Code (IaC) pour garantir la reproductibilité parfaite du déploiement. Contrairement à une installation manuelle, Ansible assure que chaque environnement (développement, test ou production) est configuré de manière identique, éliminant ainsi les erreurs humaines.

Structure mise en place dans infra/ansible/ :
- inventory.ini : Définition des cibles.
- group_vars/ : Centralisation de la "Source de Vérité".
- templates/.env.j2 : Génération dynamique du fichier .env sur le serveur.

---

## 3. CI/CD (Intégration et Déploiement Continu)

Pipeline configuré via GitHub Actions (.github/workflows/ci.yml) :
1. Compilation : Validation du code Java (Maven) et Angular (NPM).
2. Vérification Docker : Build test des images.
3. Smoke Test : Lancement réel de la stack dans le runner, attente du démarrage et test de disponibilité via curl -v http://localhost:8080/api/polls.

---

## 4. Chaos Engineering (Pumba)

Nous avons intégré Pumba pour introduire volontairement des perturbations dans notre architecture. L'objectif du Chaos Engineering n'est pas de casser le système pour le plaisir, mais de découvrir ses faiblesses cachées (erreurs de timeout, manque de politiques de redémarrage) avant qu'elles ne surviennent en production.

Tests de résilience pour garantir la haute disponibilité :

### Test 1 : Crash (Auto-Healing)
- Action : Arrêt forcé du conteneur backend toutes les 60s.
- Résultat : Redémarrage automatique via la directive restart: always.
- Conclusion : Le système est capable de s'auto-réparer.

### Test 2 : Latence (Stress Test)
- Action : Injection d'un délai de 3000ms.
- Technique : Utilisation de tc (Traffic Control) injecté via Pumba.
- Résultat : L'application supporte la dégradation réseau sans crash (temps de réponse mesuré à 5.7s).

---

## Objectifs DevOps atteints
- Automatisation : Déploiement "Zero-Touch".
- Reproductibilité : Environnements identiques via Docker & Ansible.
- Résilience : Tolérance aux pannes validée par le Chaos.
- Qualité : Pipeline CI/CD avec Smoke Test.

## Choix techniques
- Docker/Compose : Standardisation et isolation des micro-services.
- Ansible : Gestion de l'Infrastructure as Code (IaC) simple et efficace.
- Nginx : Utilisation comme Reverse Proxy pour sécuriser et centraliser les flux.
- Pumba : Outil natif Docker pour le Chaos Engineering.

## 5. Monitoring (Uptime Kuma)

Pour surveiller la santé de notre architecture micro-services, nous avons intégré **Uptime Kuma**. Cet outil nous permet de :
- Surveiller la disponibilité du Frontend et du Backend en temps réel.
- Visualiser graphiquement l'impact des tests de Chaos Engineering (on voit le service passer en "Down" puis revenir en "Up").
- Mesurer le temps de réponse moyen des différents services.

Le tableau de bord est accessible sur le port **3001** une fois le projet lancé.

### Autres éléments de surveillance :
- **Logs Docker** : Centralisation des journaux d'erreurs.
- **Healthchecks** : Intégration dans Docker Compose pour garantir que les services sont réellement prêts avant de recevoir du trafic.
- **Validation CI** : Pipeline GitHub Actions qui bloque le code si les tests de disponibilité échouent.


## Conclusion
Ce projet illustre une transformation DevOps réussie. En combinant la conteneurisation, l'automatisation par Ansible et la validation par le Chaos Engineering, nous avons créé une plateforme fiable, évolutive et facile à maintenir.
