[← Back](../README.md)

# Déploiement

Afin de préparer le partage public officiel du produit, on vous demande de déployer votre application sur la plateforme Heroku.

## Environnements de déploiement

Afin de pouvoir tester l'application avant de l'offrir aux clients, on désire utiliser 2 environnements de déploiement, soit **staging** et **production**.

- **staging** représente une version de test permettant de vérifier que tout est fonctionnel et que l'intégration technologique s'est bien effectuée. Normallement, on cherche à rendre cet environnement le plus près possible de la production afin d'éviter toute surprise lors de la migration.
- **production** représente la version officielle de votre application (qui est partagée au grand public). Lorsqu'on juge que staging fonctionne bien, on migre l'application vers cet environnement.

## Déploiement automatique

1. Votre application devra déployer de façon automatique vers l'environnement staging à partir de la branche commune de votre répertoire Git, par exemple `dev` ou `main` selon votre stratégie de branchage.
2. On devrait également pouvoir (re)déployer manuellement n'importe quel commit, branche ou tag à tout moment. Vous devrez donc ajouter un *manual trigger* prenant en *input* le commit, la branche ou le tag à déployer (voir l'action officielle [`checkout`](https://github.com/actions/checkout)). **Attention** : l'option `Use workflow from` du UI ne correspond **pas** à ce critère et change seulement la version du *fichier de pipeline* et **non** du *code au complet*!.
3. Finalement, le workflow devra comprendre 2 `job` :
    - une pour compiler et exécuter les tests (**important** car Heroku *skip* les tests lorsqu'il compile l'application). Vous pouvez utiliser les mêmes étapes que pour le workflow de tests.
    - une pour déployer, qui **dépend de la job de tests**. Vous pouvez utiliser l'action [Deploy to Heroku](https://github.com/marketplace/actions/deploy-to-heroku).

⚠️ **IMPORTANT!** ⚠️

Après le déploiement, vous **devez** :

- Faire un `healthcheck` qui appelle `/health` avec **le bon URL de base** (**différent** pour les environnements staging et production).
- Faire un *rollback* si le `healthcheck` ne fonctionne pas.

L'action de déploiement proposée permet de facilement effectuer ces actions.

## Déploiement en production

Bien que Heroku permet de rapidement migrer un environnement vers un autre, on préférera le faire par l'entremise de github afin d'exécuter les tests et de mettre en place le *rollback* automatique. Vous devrez donc créer un workflow différent pour le déploiement en production, qui ne pourra s'exécuter que **manuellement**, et comprends les mêmes jobs et critères de checkout que pour staging.

## Configuration

Heroku requiert une configuration particulière afin de comprendre comment déployer votre application. La [documentation complète](https://devcenter.heroku.com/categories/java-support) devrait contenir tous les éléments importants. Assurez-vous de filtrer et adapter l'information que vous y lisez! (ex : version de Java, features non-utilisées, nom d'application, etc.). Les articles suivants sont particulièrement pertinants :

- <https://devcenter.heroku.com/articles/getting-started-with-java>
- <https://devcenter.heroku.com/articles/deploying-java>
- <https://devcenter.heroku.com/articles/java-support>

Certaines erreurs de configuration récurrantes surviennent également. Les réponses de blogs suivants pourraient vous être grandement utiles :

- <https://help.heroku.com/P1AVPANS/why-is-my-node-js-app-crashing-with-an-r10-error>
- <https://stackoverflow.com/questions/5797860/maven-noclassdeffounderror-in-the-main-thread>

Normallement, vous ne devriez **pas** avoir besoin des éléments suivants :

- Base de données avec Heroku
- Add-ons
- Gradle, Ant et autres building tools (seulement Maven est utile)
- Spring
- Le Heroku Maven Plugin

Par contre, vous devriez vous retrouver avec les éléments essentiels suivants :

- Fichier `system.properties`
- Fichier `Procfile`
- Plugins Maven (`maven-dependency-plugin` et `maven-jar-plugin`)
- Binding dynamique au port HTTP

## ⚠️ Requis additionnels (IMPORTANT!) ⚠️

Afin de tester votre déploiement, vous **devez** fournir un fichier `deploys.yml` contenant vos URLS des environnement staging et production. La structure doit être la suivante :

```yml
deploy:
  staging:
    url: <url_staging>
  production:
    url: <url_production>
```
