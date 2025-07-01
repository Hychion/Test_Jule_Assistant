#Cahier des charges – Application de scraping de véhicules d'occasion

##1. Contexte et objectifs

Le marché des véhicules d'occasion est très fragmenté : de nombreux sites listent des annonces mais ne proposent pas toujours des moteurs de recherche avancés ou une vision unifiée. L'objectif est de développer une application Python capable de collecter, normaliser et agréger ces données afin de :

Simplifier la veille pour des particuliers ou professionnels recherchant des voitures d'occasion.

Automatiser la collecte de nouvelles annonces en fonction de critères (marque, modèle, prix, kilométrage, localisation, etc.).

Notifier l'utilisateur lorsque des offres correspondant à ses critères apparaissent.

##2. Portée et périmètre

###2.1 Sites cibles (MVP)

Priorité

Domaine

Particularités

Haute

leboncoin.fr

Pagination dynamique, API interne JSON

Haute

lacentrale.fr

HTML statique partiellement, pagination regex

Moyenne

autoscout24.fr

Contenu SSR + cookies obligatoires

Moyenne

paruvendu.fr

HTML basique, volume moindre

Évolutif : architecture pensée pour ajouter de nouveaux fournisseurs facilement.

###2.2 Périmètre fonctionnel

Recherche paramétrée (via fichier YAML ou CLI) : marque, modèle, génération, fourchette de prix, kilométrage max, année min, énergie, boîte de vitesse, distance max du code postal.

Collecte multi‑sites en parallèle.

Déduplication des annonces (même URL ou même VIN) et fusion des résultats.

Export : CSV, JSON et base de données SQLite/PostgreSQL.

Système d’alertes : e‑mail et/ou webhook (Telegram/Slack).

Interface :

CLI (obligatoire) avec commandes search, schedule, show.

Dashboard web minimal (optionnel, phase 2) avec FastAPI + React.

##3. Exigences techniques

Catégorie

Exigence

Langage

Python ≥ 3.11 avec venv + pip ou poetry.

Scraping

requests/httpx pour HTML statique, playwright pour contenu dynamique JS.

Parsing

BeautifulSoup4, lxml, éventuellement parsel ou selectolax.

Conformité

Respect strict des robots.txt et CGU ; délais entre requêtes configurables.

Qualité code

Typage typing, mypy, ruff/flake8, couverture tests ≥ 80 % (pytest).

Logging

structlog ou logging standard ; niveaux DEBUG → ERROR.

Conteneurisation

Dockerfile + docker-compose (PostgreSQL, Redis pour file d’attente).

CI/CD

GitHub Actions : lint, tests, build image.

Sécurité

Pas de stockage de données personnelles, chiffrement variables sensibles (dotenv).

##4. Architecture proposée

+-----------------+        +----------------------+        +------------------+
| Scheduler/CLI   |----->  | Orchestrator (async) |----->  | Provider modules |
+-----------------+        +----------------------+        +------------------+
                                   |                             |
                              writes JSON                fetch & parse
                                   v                             v
                            +-------------+                 +-------------+
                            |  Storage    |<----------------| HTTP client |
                            | (DB + FS)   |                 +-------------+
                            +-------------+

Provider Interface : chaque site implémente fetch(criteria) -> List[Ad].

Orchestrator : planifie les appels providers, centralise erreurs, applique déduplication.

Storage Layer : SQLAlchemy ORM, tables ads, providers, searches, alerts.

##5. Non‑fonctionnel

Critère

Cible

Performance

≥ 50 annonces/minute (HTML statique) hors temps d’attente.

Scalabilité

Ajouter un nouveau provider en < 0,5 jour dev.

Maintenabilité

Complexité cyclomatique moyenne < 6.

Portabilité

Fonctionne sous Linux et macOS, x86_64 & ARM.

Résilience

Reprises sur erreur réseau, stockage état dernier scrap.

##6. Planning indicatif (MVP)

Semaine

Livrable principal

1

Spécifications détaillées validées + diagrammes → Go

2

POC scraping Leboncoin + structure projet

3‑4

Intégration LaCentrale + moteur déduplication

5

Exports CSV/JSON + notifications email

6

Couverture tests 80 % + Docker + CI/CD

7

Documentation utilisateur + mise en production

##7. Livrables

Code source GitHub (branch main + PRs).

Image Docker prête à déployer.

Base de données SQLite exemple + schéma SQL.

Guide utilisateur (Markdown + PDF).

Rapport de tests + badge couverture.

Roadmap phase 2 (dashboard web + IA pricing).

##8. Critères d’acceptation

Tous les scénarios de test pytest passent ; couverture ≥ 80 %.

Pour un jeu de critères donné, l’appli récupère ≥ 95 % des annonces visibles manuellement.

Respect CGU/robots : aucune IP bannie après 24 h de test intensif.

Temps d’exécution complet < 2 min pour 1 000 annonces.

##9. Risques & mitigation

Risque

Impact

Prob.

Mitigation

Changement DOM fournisseur

Annonces manquées

Élevée

Tests E2E + monitoring CSS selector

Blocage IP / Captcha

Stop scraping

Moyenne

Rotation proxy, délais, user‑agent pool

Limitations légales (CGU)

Litige

Faible

Audit CGU, demande d’API officielle

Augmentation volume données

Coûts stockage

Faible

Nettoyage ancien, compaction

##10. Budget (indicatif)

Développement : 35 jour‑homme.

Infrastructure : 30 €/mois (VM + proxy pool basique).

Version : 0.9 – 01/07/2025

