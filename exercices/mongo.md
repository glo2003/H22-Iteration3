[← Back](../README.md)

# Connexion à une base de données externe

Afin de conserver l'information à plus long terme, vous devez vous connecter à une base de données externe. Pour ce faire, le service en ligne MongoDB Atlas sera utilisé. celui-ci utilise une base de données basée sur des documents appelé MongoDB. L'utilisation de l'ODM (Object Document Mapper) Morphia permettra de faciliter et d'automatiser plusieurs tâches reliées à l'utilisation de la BD. Vous devrez y stocker l'ensemble de vos données.

## Installation

Voici les librairies Maven donc vous aurez besoin :

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>4.5.0</version>
</dependency>
<dependency>
    <groupId>dev.morphia.morphia</groupId>
    <artifactId>morphia-core</artifactId>
    <version>2.2.3</version>
</dependency>
```

Vous aurez également besoin de vous créer un [compte MongoDB Atlas](https://www.mongodb.com/) (gratuit). Si vous voulez tester votre application locallement, vous pouvez également [installer une instance locale de Mongo](https://docs.mongodb.com/manual/installation/). 

## Connexion

La connexion à la base de données demande d'abord la création d'un client Mongo (`com.mongodb.client.MongoClients`), puis d'un datastore Morphia (`dev.morphia.Datastore`). Les documentations à cet effet devraient suffir. 

- Documentation [MongoDB Driver](https://docs.mongodb.com/drivers/java/sync/v4.5/fundamentals/connection/connect/#std-label-connect-to-mongodb)
- Documentation [Morphia](https://morphia.dev/morphia/2.2/quicktour.html#_setting_up_morphia)

Où :

- `applyConnectionString` indique l'URL du cluster de base de données (adresse Atlas ou locale).
- `serverSelectionTimeout` et `maxConnectionIdleTime` permettent de largement réduire les temps de timeout (utilse pour le [`healthcheck`](../features/0.healtv2.md), 1 seconde au lieu de 30).
- `mongoClient`
- `mongoUrl` représente l'URL du cluster d'Atlas ou local.
  - L'URL locale est `mongodb://localhost`
  - Pour l'**URL** d'Atlas, **vous n'avez pas besoin d'indiquer la base de donnée** après le dernier `/`. On vous conseille également de **conserver les options par défaut** qui sont incluses dans l'URL, soit `?retryWrites=true&w=majority`.

## ⚠️ Configuration (IMPORTANT!) ⚠️

Puisque vous [déployez votre application](./deploy.md) sur plusieurs environnements, vous devrez naturellement utiliser des base de données différentes pour chacun! En effet, `staging` représente un *playground* pour les développeurs, où n'importe quelle donnée peut être crée sans soucis d'affecter les clients. Les données de l'environnement de `production` doivent donc être séparées. Finalement, les **tests** ne devraient **jamais** s'exécuter sur les bases de données Atlas (en ligne), mais plutôt localement, sinon ça coûtera cher! Les bases de données de test et pour `staging` peuvent donc être effacées n'importe quand sans aucun problème.

Voici donc les configurations que vous aurez besoin pour chaque environnement :

- `staging` : base de données `floppa-staging` sur Atlas
- `production` : base de données `floppa-production` sur Atlas
- `test` / `local` (non déployé, dans le CI ou poste local) : base de données `floppa-dev` locale

La gestion de ces configurations peut se faire facilement à l'aide de variables d'environnement et de valeurs par défauts.

⚠️ **IMPORTANT!** ⚠️ Vous ne devrez donc plus utiliser vos implémentation *in memory* de vos repositories, sauf afin de les utiliser comme *stubs* pour vos tests d'intégrations, si désiré.

## Morphia

Morphia permet de *mapper* vos classes Java vers des objets représentatifs simples ("documents") pour la base de données. Il sera donc primordial de respecter le même principe que pour vos réponses du UI, c'est-à-dire de créer des classes propres aux documents MongoDB qui contiendront des champs de **primitives seulement** ainsi que les annotations Morphia. La [documentation](https://morphia.dev/morphia/2.2/quicktour.html) contient beaucoup d'informations pertinantes.

## Conseils et astuces

### Modifier le timeout de connexion

Dans le cas d'une connexion non-fonctionnelle, le driver de MongoDB attendra 30 secondes avant d'échouer, ce qui peut être particulièrement long. Il est donc possible de configurer le client MongoDB à l'aide du `MongoClientSettings` builder. Afin de réduire le timeout, vous pouvez modifier les paramètres de `serverSelectionTimeout` (dans les options de `cluster`) et de `maxConnectionIdleTime` (dans les options de `connectionPool`). Vous aurez alors également besoin de configurer l'URL à l'aide du builder. Le builder supporte également les options incluses directement dans l'URL, que vous pouvez alors conserver tel que MongoDB Atlas vous fournis. Faites attention à ne pas trop réduire ce timeout, car la connexion peut parfois prendre quelques secondes.

### Healthcheck au démarrage

Parfois, la connexion à MongoDB se fait de façon *lazy*, c'est-à-dire que le driver attendra qu'une requête soit effectuée avant de vérifier l'état de la connexion. Il est donc utile de vouloir vérifier cet état à l'aide d'une requête simple (ex : `.listCollectionNames().first()`), ce qui forcera la connexion, et donc l'échec du healthcheck lors du déploiement. Il est important de faire `.first()` puisque l'itérateur retourné est lui aussi *lazy*, donc la requête n'est pas lancée tant que l'itérateur n'est pas itéré.

### Tests

Puisque la base de données est maintenant persistante, il arrivera probablement que les tests échoueront ou deviendront fautifs. Afin de corriger le problème, on peut sans problème effacer la base de données (`.dropDatabase()`) ou des collections entières (`.dropCollection()`) afin les tests qui utilisent la base de données (tests e2e, de repositories, d'intégration, etc.). D'où l'importance d'utiliser des bases de données différentes pour les environnement de déploiement!

### Protection de la base de données de production

Afin de protéger les bases de données importantes (ex : production), il est important de soignement choisir les permissions. Être le plus restrictif possible permet de :

- Prévenir les dégâts en cas de piratage
- Prévenir les erreurs de développement (ex : effaçace de la base de données de production!)
- Découper les permissions selon les rôles dans l'entreprise (ex : CTO vs. stagiaire)
