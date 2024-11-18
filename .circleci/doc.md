# Documentation des Jobs CircleCI

Cette documentation fournit un aperçu détaillé de chaque job configuré dans le flux de travail CircleCI. Chaque job est conçu pour effectuer une tâche spécifique, allant du débogage et de la configuration de l'environnement à l'exécution des tests et des déploiements. Vous trouverez ci-dessous une description détaillée de l'objectif et des étapes de chaque job, y compris les outils utilisés et leur configuration.

---

## `debug-info`
### Description
Ce job fournit des informations détaillées sur le système et l'environnement pour aider au débogage.

### Étapes
1. **Exécuter la commande de débogage** :
   - Affiche des informations système détaillées telles que l'utilisateur, le shell utilisé, les détails du système d'exploitation, le chemin actuel, et les variables d'environnement.

---

## `build-setup`
### Description
Installe les dépendances du projet via Composer et utilise la mise en cache pour accélérer les futurs builds.

### Étapes
1. **Checkout** :
   - Récupère le dernier code du dépôt en utilisant `git checkout`.
2. **Restaurer le cache** :
   - Si un cache est disponible (par exemple les dépendances de Composer), il est restauré pour accélérer l'installation.
   - Utilise la fonctionnalité de mise en cache de CircleCI pour stocker et restaurer les dépendances.
3. **Mettre à jour les dépendances** :
   - Met à jour les dépendances dans le fichier `composer.lock` en exécutant `composer update`.
4. **Installer les dépendances** :
   - Exécute `composer install` pour installer toutes les dépendances définies dans le fichier `composer.json`.
5. **Sauvegarder le cache** :
   - Sauvegarde les dépendances installées dans le cache pour les builds futurs, accélérant ainsi les étapes d'installation.
6. **Partager l'espace de travail** :
   - Partage les fichiers et les répertoires importants entre les jobs en utilisant l'espace de travail.

---

## `lint-phpcs`
### Description
Exécute PHP_CodeSniffer pour vérifier si le code respecte les normes de codage définies par un ensemble de règles.

### Étapes
1. **Attacher l'espace de travail** :
   - Attache l'espace de travail pour pouvoir utiliser les fichiers des étapes précédentes.
2. **Installer PHP_CodeSniffer** :
   - Installe `phpcs` via Composer avec la commande `composer require --dev squizlabs/php_codesniffer`.
3. **Exécuter PHP_CodeSniffer** :
   - Exécute `phpcs` avec un ensemble de règles personnalisé. Par défaut, `phpcs` utilise les règles de codage de `PSR-2`, mais un ensemble de règles spécifique peut être configuré dans un fichier `phpcs.xml` dans le projet. Cela permet de définir des règles sur la structure du code, comme la longueur des lignes, l'indentation, et d'autres conventions de codage.
   - Exemple de commande : `./vendor/bin/phpcs --standard=phpcs.xml src/`
4. **Sauvegarder les artefacts** :
   - Sauvegarde le rapport généré par `phpcs` sous forme d'artefact pour un examen ultérieur. Ce rapport contient les erreurs de codage et les avertissements.

---

## `lint-psalm`
### Description
Exécute Psalm pour effectuer une analyse statique du code PHP, détectant les bogues et autres problèmes potentiels.

### Étapes
1. **Attacher l'espace de travail** :
   - Attache l'espace de travail pour utiliser les fichiers précédemment générés.
2. **Installer Psalm** :
   - Installe Psalm via Composer avec la commande `composer require --dev vimeo/psalm`.
3. **Initialiser Psalm** :
   - Si nécessaire, génère un fichier de configuration `psalm.xml` en exécutant `vendor/bin/psalm --init`.
4. **Exécuter Psalm** :
   - Exécute `psalm` pour analyser le code et détecter les erreurs potentielles.
   - Exemple de commande : `./vendor/bin/psalm --no-progress`
5. **Vérifier la génération du rapport** :
   - Vérifie que le rapport a bien été généré et échoue le job si aucun rapport n'est créé.
6. **Sauvegarder les artefacts** :
   - Sauvegarde le rapport généré par Psalm pour une consultation ultérieure.

---

## `security-check-dependencies`
### Description
Vérifie les vulnérabilités des dépendances PHP en utilisant `local-php-security-checker`.

### Étapes
1. **Attacher l'espace de travail** :
   - Attache l'espace de travail pour accéder aux fichiers de code nécessaires.
2. **Installer le vérificateur de sécurité** :
   - Télécharge et installe `local-php-security-checker` via Composer avec la commande `composer require --dev robertpustulka/local-php-security-checker`.
3. **Exécuter le vérificateur de sécurité** :
   - Exécute `local-php-security-checker` pour analyser les dépendances PHP et détecter les vulnérabilités.
   - Exemple de commande : `./vendor/bin/local-php-security-checker`
4. **Sauvegarder les artefacts** :
   - Sauvegarde le rapport généré (format JSON) pour une consultation ultérieure.

---

## `test-phpunit`
### Description
Exécute des tests unitaires à l'aide de PHPUnit.

### Étapes
1. **Attacher l'espace de travail** :
   - Attache l'espace de travail pour utiliser les fichiers des étapes précédentes.
2. **Vérifier la présence de tests PHPUnit** :
   - Si aucun test PHPUnit n'est trouvé, le job est ignoré.
   - Cela est généralement vérifié en recherchant des fichiers de test dans le répertoire `tests/` ou un répertoire spécifié.
3. **Installer PHPUnit** :
   - Installe PHPUnit via Composer avec la commande `composer require --dev phpunit/phpunit`.
4. **Exécuter PHPUnit** :
   - Exécute les tests dans la suite `Unit` en utilisant la commande `./vendor/bin/phpunit`.

---

## `lint-phpmd`
### Description
Utilise PHP Mess Detector pour détecter les problèmes potentiels dans le code, tels que la complexité du code ou les violations des bonnes pratiques de conception.

### Étapes
1. **Attacher l'espace de travail** :
   - Attache l'espace de travail pour accéder aux fichiers générés précédemment.
2. **Installer PHP Mess Detector** :
   - Installe PHP Mess Detector via Composer avec la commande `composer require --dev phpmd/phpmd`.
3. **Exécuter PHP Mess Detector** :
   - Exécute PHP Mess Detector pour analyser le code à la recherche de problèmes de conception ou de complexité.
   - Exemple de commande : `./vendor/bin/phpmd src/ text phpmd.xml`
4. **Sauvegarder les artefacts** :
   - Sauvegarde le rapport généré par PHP Mess Detector pour une consultation ultérieure.

---

## `lint-php-doc-check`
### Description
Vérifie la présence de documentation PHP manquante ou incomplète dans le code source.

### Étapes
1. **Attacher l'espace de travail** :
   - Attache l'espace de travail pour utiliser les fichiers des étapes précédentes.
2. **Installer PHP Doc Check** :
   - Installe `php-doc-check` via Composer avec la commande `composer require --dev php/doc-check`.
3. **Exécuter PHP Doc Check** :
   - Exécute `php-doc-check` pour vérifier si des fichiers PHP manquent de documentation ou si la documentation est incomplète.
4. **Sauvegarder les artefacts** :
   - Sauvegarde le rapport généré par `php-doc-check`.

---

## `metrics-phpmetrics`
### Description
Génère des métriques de code à l'aide de PHP Metrics.

### Étapes
1. **Attacher l'espace de travail** :
   - Attache l'espace de travail pour utiliser les fichiers précédemment générés.
2. **Installer PHP Metrics** :
   - Installe PHP Metrics via Composer avec la commande `composer require --dev phpmetrics/phpmetrics`.
3. **Exécuter PHP Metrics** :
   - Exécute PHP Metrics pour générer un rapport HTML avec diverses métriques de code, comme la couverture de tests, la complexité cyclomatique, etc.
4. **Sauvegarder les artefacts** :
   - Sauvegarde le rapport généré par PHP Metrics.

---

## `build-docker-image`
### Description
Construit et pousse une image Docker vers le GitHub Container Registry (GHCR).

### Étapes
1. **Checkout** :
   - Récupère le code à partir du dépôt en exécutant `git checkout`.
2. **Configurer Docker distant** :
   - Configure un environnement Docker distant pour la construction des images avec `docker setup`.
3. **Construire et pousser l'image** :
   - Se connecte à GHCR, construit une image Docker avec des tags spécifiques, et la pousse vers le registre avec la commande `docker build` et `docker push`.

---

## `deploy-ssh-staging`
### Description
Déploie le code dans un environnement de staging via SSH.

### Étapes
1. **Déployer via SSH** :
   - Se connecte à un serveur de staging via SSH et déploie les fichiers en utilisant des commandes comme `scp` ou `rsync`.
2. **Vérifier le déploiement** :
   - Effectue des vérifications post-déploiement pour assurer que tout fonctionne comme prévu.

---

## Workflow
Le flux de travail principal inclut tous les jobs, les exécutant dans l'ordre spécifié pour garantir un bon fonctionnement des opérations CI/CD.
