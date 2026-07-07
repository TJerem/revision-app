# Compte-Rendu de TP 8 — Révisions & Ouverture au cloud

---

## Partie 0 — Mise en place du projet fil rouge

### Question 1 : Pourquoi utilise-t-on gunicorn en production plutôt que le serveur intégré de Flask (app.run()) ?

Le serveur de développement intégré de Flask (lancé par `app.run()`) n'est pas conçu pour supporter la charge d'un environnement de production. Il est mono-processus et mono-thread par défaut, ce qui signifie qu'il ne peut traiter qu'une seule requête à la fois, bloquant les autres utilisateurs. De plus, il n'est ni optimisé pour les performances ni sécurisé contre les attaques (par exemple, les dénis de service).
En revanche, **Gunicorn** est un serveur WSGI de production robuste et performant. Il utilise un modèle de pré-forking (plusieurs processus de travail / workers) pour traiter de nombreuses requêtes de manière concurrente et parallèle, offrant une haute disponibilité, une gestion intelligente des ressources et une excellente tolérance aux pannes.

---

## Partie 1 — Révision Git & collaboration

### Question 2 : Quelle est la différence entre un merge --no-ff et un merge fast-forward ? Pourquoi préfère-t-on --no-ff pour intégrer une branche de fonctionnalité ?

*   **Merge fast-forward (avance rapide) :** Se produit lorsque la branche de destination (ex: `main`) n'a pas divergé depuis la création de la branche de fonctionnalité. Git se contente alors de déplacer le pointeur de la branche `main` directement vers le dernier commit de la branche de fonctionnalité, sans créer de commit de fusion spécifique.
*   **Merge `--no-ff` (non fast-forward) :** Force la création d'un nouveau commit de fusion (merge commit) dans l'historique, même si un simple fast-forward est possible.
*   **Pourquoi on le préfère :** Le mode `--no-ff` permet de conserver une traçabilité claire dans l'historique de Git. Il groupe tous les commits individuels de la fonctionnalité au sein d'une branche logique visible graphiquement, fermée par un commit de fusion explicite. Cela facilite grandement l'identification des fonctionnalités introduites, simplifie les audits et permet d'annuler (revert) l'intégralité d'une fonctionnalité en une seule commande en annulant son commit de fusion.

### Question 3 : Selon le Semantic Versioning (MAJOR.MINOR.PATCH), quel numéro incrémenter dans chacun de ces cas : (a) correction d'un bug, (b) ajout d'une route sans casser l'existant, (c) changement du format de réponse JSON ?

*   **(a) Correction d'un bug :** Incrément du numéro de **PATCH** (ex: `1.0.0` -> `1.0.1`), car il s'agit d'un correctif rétrocompatible.
*   **(b) Ajout d'une route sans casser l'existant :** Incrément du numéro de **MINOR** (ex: `1.0.0` -> `1.1.0`), car il s'agit d'une nouvelle fonctionnalité rétrocompatible.
*   **(c) Changement du format de réponse JSON :** Incrément du numéro de **MAJOR** (ex: `1.0.0` -> `2.0.0`), car ce changement rompt la compatibilité avec les clients consommant l'API (breaking change).

---

## Partie 2 — Révision Intégration Continue (CI)

### Question 4 : Pourquoi place-t-on les étapes de formatage et de lint avant les tests dans le pipeline ? Comment appelle-t-on ce principe ?

*   **Pourquoi :** Les étapes de formatage (Black) et de linting (Ruff) sont statiques et extrêmement rapides à exécuter (quelques millisecondes). Les tests unitaires et de couverture (Pytest) sont dynamiques et beaucoup plus lents car ils nécessitent l'exécution du code. S'il y a un défaut de style, une syntaxe invalide ou une variable inutilisée, il est préférable de bloquer le pipeline immédiatement sans faire tourner les tests unitaires plus lourds.
*   **Nom du principe :** Ce principe s'appelle le **« Fail-Fast »** (échouer rapidement).

### Question 5 : Que fait l'option --cov-fail-under=80 de pytest, et quel est l'intérêt d'un tel seuil dans la CI ?

*   **Que fait-elle :** Elle indique à `pytest` de renvoyer un code d'échec (exit code non nul) et de faire échouer le job si la couverture globale du code par les tests unitaires est strictement inférieure à 80%.
*   **Intérêt dans la CI :** Sert de **Quality Gate** automatique. Il garantit qu'aucune modification de code n'est acceptée en production si elle n'est pas accompagnée d'un niveau suffisant de tests unitaires, protégeant ainsi l'application contre les régressions et la dette technique.

---

## Partie 3 — Révision Qualité & Sécurité

### Question 6 : Distinguez en une phrase chacun de ces 4 outils : un formatter (black), un linter (ruff), un outil de SAST (bandit) et un outil de SCA (pip-audit).

*   **Black (formatter) :** Réécrit automatiquement la mise en page et l'indentation du code source pour le conformer de manière uniforme à un standard esthétique (PEP 8).
*   **Ruff (linter) :** Analyse le code source pour y détecter des erreurs de programmation, de logique courantes ou des non-respects des bonnes pratiques de style.
*   **Bandit (SAST) :** Analyse statiquement le code Python à la recherche de faiblesses et de failles de sécurité potentielles (telles que des injections ou des configurations non sécurisées).
*   **pip-audit (SCA) :** Analyse la liste des dépendances installées (requirements) pour identifier des vulnérabilités connues (CVE) référencées dans des bases de données publiques.

### Question 7 : Un secret (clé API) a été commité par erreur puis poussé. Quelles sont les étapes correctes pour réagir ?

1.  **Révoquer et renouveler immédiatement le secret :** C'est l'action prioritaire absolue car le secret doit être considéré comme compromis dès sa publication.
2.  **Nettoyer l'historique Git local et distant :** Utiliser des outils spécialisés comme `git-filter-repo` ou `BFG Repo-Cleaner` pour supprimer définitivement le fichier ou la ligne contenant le secret de toute l'historique des commits (un simple commit de suppression ne suffit pas car le commit initial contenant le secret reste accessible).
3.  **Forcer la poussée (force push) :** Mettre à jour la branche distante (`git push --force`) pour appliquer le nettoyage.
4.  **Configurer des variables d'environnement ou un gestionnaire de secrets :** Remplacer le secret par une variable d'environnement ou utiliser un gestionnaire de secrets sécurisé (Vault, GCP Secret Manager) et s'assurer que le fichier sensible est bien répertorié dans le `.gitignore`.

---

## Partie 4 — Révision Livraison Continue & artefacts

### Question 8 : On dit qu'un artefact doit être « construit une seule fois, déployé partout » (build once, deploy everywhere). Pourquoi ne faut-il pas reconstruire l'image entre staging et production ?

Reconstruire l'image Docker à chaque étape (staging puis production) présente le risque d'introduire des dérives (drifts) : des mises à jour silencieuses de packages externes, de dépendances du système d'exploitation ou des variations de configuration lors du build peuvent générer deux images différentes pour le même code.
En construisant l'image une seule fois en début de pipeline, puis en propageant cette image exacte (identifiée par son hash unique SHA) d'un environnement à l'autre, on s'assure que ce qui est déployé et s'exécute en production est à 100% identique à ce qui a été testé et validé en staging.

### Question 9 : Pourquoi ne faut-il jamais déployer le tag :latest en production ?

Le tag `:latest` est un pointeur dynamique et changeant. Si une nouvelle image est poussée avec le tag `:latest` sur le registre, toute opération automatique (comme une mise à l'échelle/scaling ou le redémarrage d'un conteneur en panne) téléchargera la nouvelle image sans aucune validation de la CI/CD. Cela rompt la reproductibilité du déploiement, rend les rollbacks extrêmement complexes et peut introduire des régressions critiques de manière totalement silencieuse.

---

## Partie 5 — Révision Documentation

### Question 10 : Citez 3 avantages concrets de la « Documentation as Code » (documentation versionnée avec le code).

1.  **Synchronisation :** Les modifications de documentation sont écrites dans les mêmes branches et Pull Requests que le code associé, garantissant que la documentation est toujours à jour avec l'application.
2.  **Revue collaborative :** Les développeurs et relecteurs valident et commentent les mises à jour de documentation en même temps que le code source directement sur GitHub.
3.  **Publication automatisée :** Un pipeline CI/CD valide la syntaxe et les liens (ex: `mkdocs build --strict`) et publie automatiquement le site final (ex: sur GitHub Pages) sans aucune action manuelle.

---

## Partie 6 — Ouverture au cloud : Infrastructure as Code

### Question 11 : Qu'apporte l'Infrastructure as Code (Terraform) par rapport à la création manuelle des ressources dans la console web du fournisseur cloud ?

L'IaC avec Terraform apporte :
*   **Reproductibilité :** Possibilité de reconstruire l'intégralité d'un environnement cloud (dev, staging, prod) à l'identique en une seule commande.
*   **Historisation et Audit :** La configuration de l'infrastructure est stockée sous forme de code Git, permettant de tracer qui a modifié quelle ressource, quand et pourquoi.
*   **Détection des dérives (Drift Detection) :** Terraform compare en permanence l'état réel du cloud avec le code local (via son fichier d'état `.tfstate`) pour repérer et corriger les modifications manuelles non souhaitées.
*   **Destruction propre :** Terraform sait exactement quelles ressources dépendent des autres et permet de tout détruire proprement (`terraform destroy`) sans laisser de ressources orphelines facturées.

---

## Partie 7 — Ouverture au cloud : Kubernetes & GitOps

### Question 12 : Cloud Run et Kubernetes déploient tous deux des conteneurs. Dans quels cas choisir l'un plutôt que l'autre ?

*   **Choisir Google Cloud Run (Serverless) :** À privilégier pour les applications web simples, les microservices, et les APIs HTTP. Il offre une simplicité absolue (pas de serveurs ni d'orchestrateur à gérer), un autoscaling ultra-rapide jusqu'à zéro instance (pas de coûts si pas de trafic) et un coût opérationnel très faible.
*   **Choisir Kubernetes (ex: GKE, AKS) :** Indispensable pour les architectures complexes multi-conteneurs (Pods complexes avec sidecars spécifiques), les besoins d'intégration réseau poussés (Service Mesh, Ingress complexes), la persistance de données lourde (StatefulSets, volumes partagés persistants), ou lorsque l'entreprise cherche une solution d'orchestration multi-cloud indépendante d'un fournisseur unique.

### Question 13 : Qu'est-ce que le GitOps et quel est son principal intérêt ?

*   **Définition du GitOps :** C'est une démarche d'automatisation des déploiements applicatifs et d'infrastructure où un dépôt Git sert d'unique source de vérité déclarative. Un agent installé dans le cluster (ex: ArgoCD ou FluxCD) compare en continu l'état déclaré dans Git avec l'état réel du cluster et applique automatiquement les corrections nécessaires pour les synchroniser.
*   **Principal intérêt :** Garantit une cohérence absolue de la production avec Git, empêche les dérives de configuration manuelles (reconciliation automatique), simplifie la sécurité (pas besoin de donner des accès en écriture sur le cluster aux développeurs ou à la CI, c'est l'agent interne qui tire le code), et rend les retours en arrière (rollbacks) aussi simples qu'un `git revert`.

---

## Partie 8 — Quiz de synthèse & recherche autonome

### Question 14 : Présentez le service de conteneurs managés du fournisseur choisi : nom, principe, 2 points communs et 2 différences avec Cloud Run. Indiquez la source (lien de la doc consultée).

*   **Fournisseur cloud choisi :** Microsoft Azure
*   **Nom du service :** Azure Container Apps (ACA)
*   **Principe :** ACA est un service Serverless entièrement managé permettant de déployer et de mettre à l'échelle des applications conteneurisées et des microservices sans avoir à gérer la complexité d'un cluster Kubernetes sous-jacent. Il repose sur des standards open-source (Kubernetes AKS, KEDA pour l'autoscaling et Dapr pour la gestion des microservices).
*   **2 points communs avec Google Cloud Run :**
    1.  **Facturation serverless à l'usage et mise à l'échelle à zéro :** Les deux services adaptent automatiquement le nombre d'instances de conteneurs de 0 à des dizaines en fonction du trafic HTTP entrant, et ne facturent rien en l'absence d'activité.
    2.  **Abstraction de l'infrastructure :** L'hébergement, la mise à jour de l'OS hôte et l'orchestration des conteneurs sont totalement gérés et masqués par le fournisseur de cloud.
*   **2 différences avec Google Cloud Run :**
    1.  **Environnement et regroupement d'applications (Environnement Container Apps) :** ACA permet nativement d'exécuter plusieurs conteneurs au sein d'un même groupe d'applications (équivalent d'un Pod Kubernetes) partageant le même réseau et stockage local, tandis que Cloud Run déploie un seul conteneur principal par service (bien qu'il supporte de simples sidecars plus limités).
    2.  **Intégration d'outils cloud-native avancés :** ACA intègre directement **KEDA** (Kubernetes Event-driven Autoscaling) pour réaliser de la mise à l'échelle basée sur des événements (ex: taille d'une file de messages RabbitMQ ou Azure Service Bus) et **Dapr** pour structurer des microservices (gestion d'état distribué, appels inter-services sécurisés), alors que Cloud Run s'appuie principalement sur Knative et requiert d'autres configurations GCP pour ces fonctionnalités.
*   **Source :** [Documentation Azure Container Apps](https://learn.microsoft.com/fr-fr/azure/container-apps/)