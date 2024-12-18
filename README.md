# Cours Openshift 

### Users

Sur la Sandbox, il n'st possible d'avoir recourt qu'à un seul utilisation, qui est automatiquement nommé `developer`. De base, il est cependant possible de se retrouver à travailler avec plusieurs utilisation. Dans ce cas de figure, alors on peut utiliser la commande `oc login` pour réaliser une connexion plus classique à base de credentials. Dans notre cas, la commande sera fournie par la sandbox et contiendra un token et un URL.

---

### Project

Lorsque l'on travaille avec Openshift, il est fréquent d'avoir besoin de créer des projets. En créant un projet, on peut compartimenter nos déploiements et nos ressources openshift de sorte à éviter les confusions. 
* Pour créer un projet, on dispose de la commande `oc new-project <project-name>`
* Si l'on veut lister les projets auquel nous avons accès, on peut utiliser `oc projects`
* Pour passer d'un projet à l'autre, on utilisera `oc project <name>`

---

### Pods

Dans l'écosystème Kubernetes, un pod est une abstraction de conteneur, il permet de faire tourner un ou plusieurs conteneurs. Il est possible, dans openshift, de voir la documentation d'un élément, ainsi que les composants d'un fichier YAML de configuration pour cet élément, avec la commande `oc explain <ressource-type>`. Pour un pod, on utiliserait donc `oc explain pod`. Il est même possible d'aller plus loin et de demander l'explication des sous parties du fichier de configuration YAML via oc explain `pod.spec.<section-name>`.

---

### Créer un élément Openshift

Lorsque l'on veut créer un élément dans notre cluster, le plus simple est la rédaction d'un fichier YAML et son envoi à Openshift. Pour envoyer un fichier YAML, on passe par la commande `oc apply -f <chemin-du-fichier>` ou pas `oc create -f <filepth.yaml>` comme on le ferait avec K8s. Une fois la commande réalisée, la vérification passera, comme souvent, par la commande `oc get <type>`. 

Pour aller un peu plus loin, on peut lancer un terminal dans notre pod et y executer des commandes. Pour cela, on va utiliser `oc rsh <pod-name>` (pour quitter, utiliser ensuite la commande `exit`).

---

### Supprimer les élément Openshift

Une fois créé et utilisé, un élément Openshift peut devenir inutile. Il est alors possible de le retirer via la commande `oc delete <ressource-type>`. Il est possible d'ajouter une sélection via un tag (et `-l`)ou de spécifier que l'on veut tout supprimer avec l'option `--all`

---

### YAML

Le format **YAML** est le format par défaut des fichiers d'entités Kubernetes, et donc de la même façon, d'Openshift. Il s'agit, comme pour le format **JSON**, d'un ensemble clés-valeurs. 

```yaml
# Pour faire des ensemble clés-valeurs classiques
key1: value1
key2: value2

# Pour faire des liste d'éléments
elements: 
  - 1
  - 2
  - 3

# Pour une liste de clés-valeurs multiples
elements:
  - element1-key1: value1
    element1-key2: value2
  - element2-key1: value1
    element2-key2: value2
```

--- 

### DeploymentConfig

Un DeploymentConfig est une ressource spécifique à Openshift servant à administrer plusieurs pods. Ils définissent l'objectif d'un déploiement et l'état que l'on aimerait atteindre à l'issue de ce dernier. On y retrouve par exemple combien de replicas on aimerait avoir pour nos pods. 

Pour créer un DeploymentConfig rapidement, on peut utiliser la commande `oc new-app <image-tag>`, par exemple `oc new-app antoineutopios/demo-openshift`. Openshift va créer par défaut un **ImageStreamTag** pour garder une trace de l'image que l'on vient de lui envoyer. Le déploiement sera ensuite créé, de même qu'un service. Par défaut, un service ne donnera pas l'accès à l'extérieur du cluster. Pour cela, il nous faudra une **route**. Il est possible d'observer notre déploiement via la commande `oc status`.

Un pod créé via un DeplymentConfig va se retrouver attribué un nom suffixé d'un ID aléatoire de par la nature d'un DeplomentConfig. En effet, de base, il est censé servir à définir un listing de pod, et non forcément un pod unique. 

Pour lister les DeploymentConfig, on peut utiliser `oc get dc`.

---

### Faire le nettoyage

Dans notre cluster, il se peut que l'on finisse rapidement par avoir des dizaines de ressources tournant en même temps. Celles-ci peuvent devenir obsolètes et nécessiter une suppression. Dans ce cas de figure, le plus simple est de passer par une commande de suppression telle que `oc delete <type>`. En utilisant les tags, la commande devient encore moins rébarbative et permet de supprimer tout type de ressource lié à notre application: `oc delete all -l app=<app-name>`. Pour obtenir les tags de notre application, on peut utiliser la commande `oc describe <entity>`.

---

### Nommer les déploiements

Dans le cas où l'on aurait besoin de donner un nom personalisé à notre application créée via `oc new-app`, on peut ajouter l'argument `--name <nom-custom>`. On peut vérifier que les éléments ont été créé avec le bon label via `oc status`, encore une fois.

---

### Déployer à partir d'un repo Git 

La manipulation nécéssaire au déploiement d'une application via Git est la même que celle d'un déploiement à partir d'une image Docker. Il suffit de remplacer l'adresse de l'image par une adresse de repo Git et le tour est joué !

Ainsi, notre commande de tout à l'heure peut se voir être transformée en `oc new-app https://github.com/antoine-utopios/openshift-formation.git --as-deployment-config`. Cette fois-ci, cependant, un build aura lieu et donnera un BuildConfig. 

Il est possible de l'observer via la commande `oc logs -f <chemin-buildconfig>` tel que `oc logs -f bc/demo-app`. Si l'on observe notre déploiement avec de nouveau la commande `oc status`, alors on va pouvoir remarquer que l'on possède, comme précédemment, un tag pour sélectionner par la suite notre application. 

---

### ReplicationController

Une autre entité présente dans Openshift est le **ReplicationController**. Cet élément est utilisé par notre **DeploymentConfig** pour contrôler le nombre de pods de notre application. Pour les observer, on peut utiliser la commande `oc get -o yaml <ressource>`. Par exemple, on peut lire le fichier YAML de notre déploiement généré par Openshift en réalisant la commande `oc get -o yaml dc/hello-world`. Ce fichier est semblable à celui d'un Pod, mais il présente quelques particularités: 
* **replicas**: Le nombre de replicas que notre ReplicationController doit créer. Par défaut, cette valeur sera de 1.
* **template**: Cette section va définir le contenu du pod. L'ensemble de la template va se voir être répliquer X fois par le ReplicationController à l'issue du déploiement. Cette section contient par exemple le lien vers notre image Docker.

Dans le cas où l'on voudrait mettre à jour notre application, le DeploymentConfig va créer un nouveau ReplicationController générant de nouveaux pods. Une fois les pods créé, le traffic va être redirigé vers ce nouveau ReplicationController tandis que l'ancien sera libéré. Ce mode de fonctionnement est bien entendu configurable.

Pour observer les différents ReplicationController de notre application, on peut utiliser `oc get rc`.

---

### Rollout et Rollback

Lorsque l'on veut obtenir la nouvelle version de notre application de façon manuelle, on a recourt au mécanisme du Rollout. Ce mécanisme peut être trigger par l'utilisation d'une commande. `oc rollout latest dc/<name>` va causer la création d'un nouveau pod. Ce pod va chercher la dernière version de notre application tandis que d'autres pods vont se charger du déploiement. Une fois récupéré et le déploiement effectué, on voit apparaitre des pods contenant notre nouvelle version de l'application tandis que les anciens pods sont petits à petit libérés. En fin de ce processus, les pods de déploiement de la nouvelle version seront libérés. 

Si l'on tiens à faire le chemin inverse, c'est égalemnt possible. Pour cela, on va utiliser la commande `oc rollback` de sorte à récupérer l'ancienne image de notre déploiement et la remettre en place. Afin d'éviter que le processus de mise à jour automatique des image ne ramène une image plus récente sans que l'on en ait conscience, les triggers du déploiements se voient être désactivés. Pour les réactiver, il nous faudra le faire manuelle via la commande `oc set triggers dc/<name> --auto`. 

---

### Networking

Tout comme les pods, les services sont un composant qu'Openshift emprunte à Kubernetes. Il est possible d'en lire la définition avec la commande `oc explain svc`. Un service est une abstraction de service logiciel, ce qui fait que l'on peut avoir un service reliée à plusieurs éléments de notre cluster. Il dépend d'un proxy (IP et port) et d'un sélecteur. Un sélecteur se sert d'un label pour permettre l'accès à notre application.

---

### Crréation d'un service

Pour créer un service, il nous faut simplement utiliser la commante d'exposition de notre élément. Pour exposer un pod, cette commande pourrait alors ressembler à `oc expose -- port XXXX pod/<pod-name>`. Via cette commande, on peut accéder à un port particulier de notre pod. Pour pouvoir tester notre service, en attendant d'avoir des routes, il nous est possible de créer un nouveau pod qui va interroger le premier et de faire une requête à partir de ce dernier (`wget -qO- <pod-link:<port>>` une fois connecté au pod avec `oc rsh <pod-name>`). 

Pour éviter d'avoir à retenir les adresses IP des pods alors qu'elles peuvent être amenées à changer, on peut utiliser directement le nom du service pour intéroger l'un de nos pods concernés, par exemple `wget -qO- <service-name>:<port>`. 

---

### Exposer une route

Si l'on veut maintenant pouvoir accéder à notre pod à l'extérieur de notre cluster, il va nous falloir posséder une route. Pour créer une route, il nous faut d'abord un service, la route elle seule ne suffira pas et n'est pas un remplacement pour un service. Elle arrive en complément de ce dernier. Pour en obtenir une, il nous suffit d'utiliser la commande `oc expose` sur un service. Pour ensuite voir notre route, on peut utiliser la commande `oc status`.

---

### Définitions de routes

Ne faisant pas partie de Kubernetes mais de Openshift, la meilleure source de documentation est celle de RedHat. Il s'agit cependant de serveur DNS permettant, via une adresse url, d'accéder à notre cluster et au service voulu depuis le monde extérieur. Pour permettre la redirection vers notre élément, Openshift utilise un service nommé **Nip.io**. Par défaut, le sous-domaine se trouve être le nom du service suivi du nom du projet et de l'IP / URL de l'emplacement cluster.

---

### ConfigMaps

Un autre composant qu'Openshift emprunte à Kubernetes est la **ConfigMap**. Cet élément permet de stocker des configuration consomable par les pods. Il peut être intéressant de nous en servir lorsque l'on doit déployer notre application sur plusieurs environnements. De la sorte, on peut stocker les variables d'environnements de l'environnement de développement indépendamment de celle de production, et référencer simplement la ConfigMap voulu en fonction de notre déploiement. L'ensemble de clés restant le même, notre déploiement ira simplement piocher les valeurs associées en fonction de quel fichier de configuration on lui a donné. Pour créer la liaison entre les deux éléments, il suffit simplement de **référencer** la ConfigMap au niveau des pods que l'on veut alimenter.

Il faut cependant faire attention, tout élément présent dans notre cluster peut accéder et lire dans la ConfigMap. Elle n'est donc pas à utiliser pour les données sensibles. Pour cela, une autre ressource sera préférée, le **Secret**. L'autre limitation de la ConfigMap est la taille: Elle est limitée à 1MB.

---

### Créer une ConfigMap

Pour créer une ConfigMap, il va encore une fois falloir utiliser la commande `oc create`. A titre d'exemple, cette fois-ci, nous n'utiliserons pas de fichier YAML, mais les les arguments et les options disponibles dans cette commande: `oc create configmap message-map --from-literal MESSAGE="Hello from ConfigMap"`. Pour en vérifier ensuite le contenu, il suffit d'ouvrir son fichier YAML avec `oc get -o yaml cm/<name>`.

---

### Consommer notre ConfigMap

Nous allons vérifier ensuite que notre ConfigMap est disponible pour nos pods. Pour cela, nous allons utiliser l'image de base `hello-world` car elle va chercher une variable d'environnement `MESSAGE` par défaut. Pour lier la ConfigMap à notre déploiement, il va falloir utiliser la commande `oc set env dc/<name> --from cm/<name>` tel que `oc set env dc/hello-world --from cm/message-map`.

---

### ConfigMap à partir de fichier

Si l'on l'avait voulu, on aurait également pu créer une ConfigMap à partir d'un fichier. Pour cela, la commande `oc create configmap <name> --from-file=<filepath>`. Ce genre de ConfigMap possède comme clé le nom du fichier et comme valeur son contenu, mais il est possible d'étendre la commande précédente pour nomme la variable qui va être peuplée: `oc create configmap <name> --from-file=<VARIABLE_NAME><filepath>`.

---

### ConfigMap à partir de dossier

Il est également possible de peupler une ConfigMap à partir d'un dossier entier. Pour cela, il suffit, en lieu et place d'un chemin de fichier, de donner en argument un chemin de dossier: `oc create configmap <name> --from-file=<directory-path>`. Nous pouvons encore une fois vérifier le contenu de la ConfigMap via la commande `oc get -o yaml cm/<name>`.

---

### Les Secrets

Lorsque l'on veut travailler avec des données sensibles, il est plus intéressant de travailler non pas avec des **ConfigMap** mais plutôt des **Secrets**. Il existe plusieurs types de secrets: 

* Les secrets dits **opaques**, qui peuvent contenir un ensemble de cés arbitraires et de valeurs, mais n'ayant pas forcément de volontée d'intégration avec Openshift. Ils peuvent être intéressant cependant pour une chaine de connexion à une base de données. 
- Les secrets de **jeton de service compte** qui vont service principalement à s'authentifier vis à vis des APIs d'Openshift. Par défaut, le système d'intégration d'Openshift va restreindre l'accès à ce type de secret. Il faudra donc, manuellement, en accorder l'accès aux utilisateurs.
- Les secrets d'**identifiants Docker** servent quant à eux principalement à permettre l'accès de registres privés d'images Docker, que cela soit pour en récupérer ou en pousser. 
- Les secrets d'**authentification basiques**, servant à s'authentifier lorsqu'il s'agit d'un type d'authentification du genre **Basic**, **SSH** ou **TLS**. 

---

### Créer un secret

Lorsque l'on souhaite créer un secret, il nous faut utiliser encore une fois la commande `oc create` tel que `oc create secret <type> <name> --from-literal <VARIABLE_NAME>="value"`. Il est ensuite possible, encore une fois, d'observer le contenu du fichier YAML généré par Openshift via la commande `oc get -o yaml`, mais l'on remarquera cette fois ci que le texte de la valeur à été encodé pour éviter les audacieux. 

--- 

### Utiliser le secret

Pour nous servir de notre secret, on peut brancher des variables d'ennvironnement de la même façon que l'on le faisait avec notre ConfigMap précédemment. Par exemple `oc set env dc/hello-world --from secret/message-secret`.

---

### Les Images

Pour gérer les images, Openshift possède plusieurs solutions. En effet, il est possible, via Openshift, d'opérer des intéraction complexes à base d'images, ce gràace aux outils fournis par RedHat. Les deux ressources principales que l'on est amené à utiliser sont les **ImageStream** et les **ImageStreamTag**. Ces deux ressources peuvent s'apparrenter au **nom** et aux **tags** d'une images sur un registre de conteneur, respectivement. L'utilisation des ImageStreamTags permet de mettre à jour automatiquement les déploiement grâce au **DeploymentConfig**. Si l'on observe nos ImageStream via la commande `oc get is`, on remarque facilement que le comportement par défaut de création de cet entité passe par le système du `latest` au niveau du tag de l'image. Si l'on veut observer la version longue de cet affichage, avec les tags, on peut utiliser `oc get istag`.

---

### Créer un ImageStream

Un ImageStream va se créer en réalisant un import d'une image depuis un registre de conteneur. Par exemple, si l'on ajoute une image via la commande `oc import-image --confirm <image-path>`, on va créer automatiquement un ImageStream et un ImageStreamTag que l'on peut vérifier avec `oc describe istag <name>`. Dans le descriptif se trouveront les différentes variables d'environnement que l'image utilise, l'ID de l'image et donc le lien vers l'image sur laquelle notre ImageStream pointe dans le registre distant.

Une ImageStream va servir principalement lorsque l'on veut déployer une application à nouveau en se basant sur l'image que l'on vient d'importer. Par exemple, `oc new-app` va nous permettre d'aller directement piocher dans l'image importée au lieu de redemander un nouvel import. Pour le nom de l'image, elle se trouve être le nom de notre projet suivi de son ImageStream tel que, par exemple: `oc new-app <project-name>/<image-name> --as-deployment-config --name <name>`.

---

### Importer plus de tags

Si l'on a besoin d'avoir accès à plusieurs tags pour notre image, alors on peut utiliser la commande `oc tag <original> <destination>` permettant de copier des tags depuis une origine vers par exemple notre propre cluster. Par exemple, pour avoir un tag supplémentaire pour notre dummy image, on peut faire `oc tag antoineutopios/demo-openshift:update-message hello-world:update-message`. L'import peut ensuite être vérifié via la commande `oc get is` et le visionnage de la colonne **TAGS**.

---

### Images privées

Si l'on veut travailler avec une image privée, il va falloir changer un peu notre mode de fonctionnement.  Au moment de push, nous aurons besoin d'alimenter notre commande via les credentials. Il en sera bien entendu de même lorsqu'il s'agira de la récupérer sur Openshift. 

On va donc commencer par se loguer sur le registre de conteneur privé avec `docker login` puis y déposer l'image via docker push `quay.io/<nom-registre>/<nom-image>`

---

### Utiliser notre image

Lorsque l'on veut accéder à une image privée, il va en résulter une erreur sur le résultat de notre commande Openshift. Pour résoudre notre erreur, il va falloir commencer par créer un **secret** contenant les identifiants: `oc create secret docker-registry demo-image-pull-secret  --docker-server=$REGISTRY_HOST --docker-username=$REGISTRY_USERNAME --docker-password=$REGISTRY_PASSWORD --docker-email=$REGISTRY_EMAIL`. On remarque qu'ici, le type de secret créé est **docker registrry**, ce qui correspond bien à notre but final.

Créer un secret n'est cependant pas suffisant, il nous faut lier notre secret aux autres de sorte à permettre sa pioche lors de la tentative d'authentification au moment du pull: `oc secrets link default demo-image-pull-secret --for=pull`. Pour vérifier ensuite la jonction, il est possible de décrire le secret via `oc describe serviceaccount/default`.

Il ne nous reste plus qu'à utiliser la même commande de création d'application que précédemment pour vérifier que tout est bien mis en place: `oc new-app <lien-repo-prive>`.

---

### Les Builds 

Dans Openshift, un build est similaire à ceux de Docker. Lorsque Openshift récupère le code source d'un repository Git, il va automatiquement réaliser un build. Pour ce faire, Openshift va utiliser une nouvelle ressource, le **BuildConfig**. Pour réaliser un build, il suffit d'utiliser la commande `oc new-build <repo-url>`. On obtiendra alors en plus de notre build les Imagestream pointant vers l'image crée ainsi que ses potentielles dépendances. On peut ensuite s'amuser à regarder le contenu de notre BuildConfig via la commande `oc get -o yaml bc/<name>`.

Dans ce fichier se trouverons plusieurs informations intéressantes telles que: 
- Dans la section **spec**:
  - **source**: Ce que Openshift doit utiliser pour réaliser notre build
  - **output**: Que faire une fois que l'image est crée
  - **strategy**: Quel mode de build on utilise, ici via docker, soit **dockerStrategy**
  - **triggers**: Ce qui peut amener à relancer un build de façon automatique. Ici, on aura par défaut le fait de mettre à jour notre image de dépendance.

Pour voir le build, on peut ensuite faire la commande `oc get build`. 

---

### Voir les logs de notre build 

Si l'on veut avoir plus de visualisation au niveau de notre build, on peut le faire via la commande `oc logs`. Cette commande va nous aider en cas de soucis avec le build de nos images: `oc logs -f bc/<name>`.

---

### Démarrer manuellement un build

Si l'on a besoin de démarrer manuellement un build dans Openshift, il est possible de le faire via une commande dédiée. Cette commande `oc start-build bc/<name>`, va causer la création de nouveaux pods dédiés au build de notre image. Il est possible de les observer en direct via la commande `oc get pods -w`. Une fois le build complété, on peut voir que notre ImageStream a été mis à jour en utilisant `oc describe is/<name>`.

---

### Annuler un Build

Dans le cas où l'on aurait besoin d'annuler un build, il est possible de le faire avec `oc cancel-build bc/<name>`. Cette commande est utile en cas de build trop long ou si l'on n'a plus besoin du résultat de ce build. 

---

### Les Webhooks 

Un Webhook est un élément clé de CI/CD, il permet de déclencher un évènement en cas de modification de notre code source. Il s'agit d'endpoint accessible via le protocole HTTP. Un repository Git est configuré de sorte à appeller cet endpoint en cas de push de code de part un développeur. Openshift, de son côté, demande aux utilisateurs de passer un jeton spécifique avec leur requête. Ce jeton sera généré automatiquement avec la création d'un DeploymentConfig ou d'un BuildConfig (`oc new-app` ou `oc new-build`, en gros) de sorte à éviter les soucis de sécurité. Lorsque l'**écouteur de Webhook** capte une requête, il va demander un nouveau build automatiquement. 

---

### Utiliser un webook manuellement

Si l'on veut provoquer nos webhooks manuellement, il nous faut posséder le jeton d'authentification ainsi que l'url du webhook. Pour obtenir le jeton secret, il est possible de le trouver dans le YAML du BuildConfig avec `oc get -o yaml bc/<name>`. Pour obtenir l'url du repo Git, on peut cette fois ci utiliser `oc describe bc/<name>`. Enfin, pour trigger le webhook, on va utiliser la commande `curl -X POST -k <url-avec-secret>` et le tour est joué !

---

### Manipuler les branches

Si l'on veut pouvoir manipuler les builds provenant de branches, alors il est possible d'ajouter, en fin du lien vers notre repo Git, la syntaxe suivante: `#<nom-branche>`. On obtient alors par exemple `oc new-build https://github.com/antoine-utopios/openshift-formation.git#update-message`. Pour vérifier, on peut utiliser `oc logs -f bc/hello-world` en fin de notre build.

---

### Sous-dossiers de build

Si l'on a besoin de créer des dossier lors d'un build, il est possible de le faire en ajoutant un argument dans la commande de build tel que: `oc new-build <url> --context-dir <dir-name>`. Pour vérifier, on peut encore une fois utiliser `oc logs -f bc/<name>`.

---

### Les Build Hooks

Iles permettent de modifier le fonctionnement des builds. Il est possible de déclencher des fonctionnalités à un moment précis du build, qui de base, se constitue de ces étapes:
- Au début du build
- Lors du build
- Une fois la complétion du build
- **Lanchement des build hooks**
- L'image est envoyée à l'ImageStream

Si l'on le veut, on peut par exemple réaliser des tests et ainsi empêcher l'ajout de l'image à l'ImageStream, par exemple.

Pour ajouter un build hook, il est possible d'avoir recourt à une commande: `oc set build-hook bc/<name> --post-commit --script="echo This is my script!"`. **post-commit** est le type de build hook que l'on cible ici, mais il est possible d'en choisir un autre parmi la liste disponible chez RedHat. Pour retirer un build hook, il faut faire la même commande que précédemment, et remplacer l'argument **script** par **remove** tel que: `oc set build-hook bc/<name> --post-commit --remove`.

---

### Source-to-Image (S2I)

Dans Openshift, il est possible d'envoyer directement le code source de notre repository **sans avoir recourt à un Dockerfile**. Pour cela, Openshift va utiliser une série de scripts pour **assembler** un équivalent d'image avant d'exécuter notre assemblage sur un pod via l'équivalent d'une commande **RUN** dans un Dockerfile.

Les avantages de cette méthode de fonctionnement est l'absence d'écriture fastidieuse de Dockerfile et leur maintient, l'expertise RedHat mise à disposition des utilisateurs d'Openshift ainsi que la possibilité d'utiliser des builder images comme par exemple Python, Ruby, NodeJS ou encore Java.

---

### Utiliser le système S2I

Le processur d'utilisation du déploiement d'applications via S2I est similaire à celui via des images. Il y a d'abord un clone du repository, puis vient l'exécution de diverses étapes présentes dans le fichier d'assemblage. Il y a une détection du langage utilisé dans notre repository par Openshift. L'exécution des commandes par Openshift se fera également en fonction des instructions présentes dans le fichier d'assemblage créé par les équipes d'Openshift. La commande de lancement de notre build est: `oc new-app <repo-url> --context-dir <sub-folder> --as-deployment-config`. 

Une fois le déploiement réalisé, Openshift va nous fournir une commande d'exposition pour pouvoir tester notre application directement.

---

### La détection du langage

La détection du langage dans notre repository est plutôt rapide. Lors de la récupération d'un repository, Openshift va tenter de chercher une image Docker pour executer la **dockerStrategy**. Si il n'en trouve pas, alors il va passer par la stratégie de déploiement **sourceStrategy**. Ces deux stratégies, rappellons le, se trouvent présentent dans le YAML de notre **DeploymentConfig**.

Lorsque Openshift va analyser le repository, il va chercher des ressemblance entre les builder et les fichiers présents. Par exemple, pour Python, il va chercher les fichiers `requirements.txt` et / ou un fichier `app.py`. Du moment que l'on respecte ces conventions, alors il est possible qu'Openshift auto-detecte notre application en tant qu'application Python.

---

### Image de Build explicite

Il est possible de déclarer nous même quelle image de build on souhaite utiliser en cas de déploiement via S2I. Pour cela, il nous faut altérer légèrement la commande de déploiement en ajoutant le type de builder image avant l'url du repo Git, séparé via un tilde (`~`) tel que: `oc new-app <builder-image>~<url-repo>`.

Bien entendu, lorsque l'on utiliser des images de build de façon explicite, il faut rester critique vis à vis du type de code que l'on essaye d'envoyer à notre déploiement.

---

### Modifier le script fourni

Si l'on a besoin de faire notre propre script de génération d'assemblage via la méthode S2I, il est possible de placer un fichier spécifique à un endroit donné de notre repository. Cet emplacement se trouve être `.s2i/bin`. Via l'utilisation de ce script, on peut adapter une image de build à nos besoin. 

Pour cela, il va falloir écrire un script tel que: 

```bash
#!/bin/bash

# Instructions à effectuer avant l'assemblage
echo "J'ai lieu avant l'assemblage !"

# Appel de l'assemblage de base
/usr/libexec/s2i/assemble

# Instructions à effectuer aprè l'assemblage
echo "J'ai lieu après l'assemblage !"
```

---

### Les Volumes

Lorsque l'on veut stocker des fichiers dans notre Pod, il va falloir utiliser un **Volume**. Pour rendre cet emplacement disponible, il va falloir ensuite monter le volume dans notre pod. Il est possible d'utiliser des sources provenant d'ailleurs, telles qu'un S
chez AWS. Dans tous les cas, le résultat final est que notre application peut manipuler ces fichiers et potentiellement les partager avec d'autres pods / applications. 

Pour pouvoir ajouter un volume à notre déploiement, il va falloir utiliser la commande suivante: `oc set volume dc/<name> --add --type <type> --mount-path /folder/structure/for/path`

---

### emptyDir

Par défaut, ce type de volume commence en étant vide. Il s'agit d'un volume de stockage temporaire qui suit le cycle de vie d'un pod. Il n'est donc pas recommandé d'utiliser ce type de volumes pour des données dont on doit assurer la persistance. La seule chose à laquelle il est résilient, ce sont les reboot d'un pod. Dans ce cas de figure, les fichiers ne seraient pas perdus. 

Pour vérifier l'ajout de notre volume, il suffit encore une fois d'ouvrir le fichier YAML résultant de la modification de notre DeploymentConfig. Pour cela, on utilise la commande `oc get -o yaml dc/<name>`. Parmi la section spec, on peut voir la section **volumeMounts** qui résulte de notre ajoute d'un volume. Celle-co contient l'ID du volume et son type.

Il est ensuite possible d'ouvrir un terminal dans notre pod et de vérifier la présence de notre volume via la commande `ls` qui va nous montrer un dossier vide.

Ce genre de volume est utile par exemple en cas de création et de stockage d'un cache par notre pod.

---

### ConfigMap en tant que volume

Lorsque l'on monte une ConfigMap en tant que volume, chaque fichier ajouté au volume proviendra d'une clé et alors que leur contenu proviendra des valeurs de la ConfigMap.

`oc create configmap cm-volume --from-literal file.txt="Je serai le contenu du fichier!"`

Pour vérifier ensuite que notre ConfigMap est bien montée en tant que volume, il va encore une fois falloir utiliser `oc rsh <nom-pod>` pour nous connecter au pod. Ensuite, on ira lister (`ls`) les fichiers présents dans le dossier spécifié en tant que mountPath et leur contenu via une commande `cat`.

---

### DeplymentConfig avancé - Triggers

Dans Openshift, via les DeploymentConfig, il est possible de créer des triggers. Ces triggers sont à la base de mécanisme d'automatisation dans notre cluster. Par exemple, via  **ImageChange**, il est possible de faire en sorte que la modification d'un ImageStreamTag cause un nouveau déploiement dans Openshift. Un autre trigger serait **ConfigChange**, qui va causer un rollout dans le cas où l'on modifierait le fichier YAML du DeploymentConfig. Dans le cas où l'on déploie une application via la commande `oc new-app`, alors les deux triggers seront automatiquement ajouté par notre environnement Openshift.

---

### Tester ConfigChange

Pour tester ce trigger, il suffit de modifier le fichier YAML de configuration de notre déploiement. Pour ce faire, on peut tout simplement ajouter un volume de type emptyDir à notre déploiement via la commande `oc set volume dc/<name> --add --type emptyDir --mount-path /folder/path`

---

### Modifier les triggers

Une fois notre trigger testé, il peut être intéressant de voir comment en retirer. Pour retirer un trigger de notre DeploymentConfig, il est possible de le faire via des commandes Openshift. Pour ce faire, on peut utiliser la commande `oc set triggers dc/<name>`. Sans flag, cette commande va afficher les triggers. Pour ajouter un trigger, on peut ajouter des flags tels que `--from-config` ou `--from-image <image-name>`. On peut specifier le conteneur qui va 
s'en charger via `-c <nom>`.

Pour en retirer, il faut ajouter `--remove` à notre commande, tel que `oc set triggers dc/<name> --remove --from-config`. Si l'objectif est maintenant de manipuler le trigger relatif à l'image, alors la commande sera `oc set triggers dc/<name> --remove --from-image <image-name>:<tag>`.

---

### Les stratégies de déploiement

Lorsque l'on fait une modification de notre déploiement et qu'une nouvelle version de notre application est à faire apparaitre dans notre cluster, alors les stratégies de déploiement peuvent avoir leur importance. Par défaut, la stratégie **Rolling** consiste à démarrer la nouvelle version, faire passer le traffic vers la nouvelle version, puis supprimer l'ancienne version. Il est aussi possible de faire appel à la stratégie **Recreate** de sorte à ce que les pods soient tous stoppés avant que l'on fasse apparaitre les nouveaux. Cette dernière requiert un downtime et peut causer des soucis vis à vis des utilisateurs, à part si l'on a une raison particulière. Enfin, il existe la stratégie **Custom** qui va demander de fournir une image de déploiement personnalisé.

---

### Les hooks de déploiement

Les deux stratégies de base peuvent également gérer des hook lors du déploiement. Ces hooks vont permettre, si l'on le désire, de faire s'exécuter des commandes et instructions avant (pre hook) ou après le déploiement (post hook). Un autre hook, le mid hook, est disponible et peut être appellé entre les deux étapes de base du déploiement, durant le moment de creux. 

Pour gérer les hooks, on peut également réagir à leur succès ou échec vis à vis de la poursuite du déploiement. Par exemple, si l'on veut utiliser **Abort**, alors le déploiement ne devra pas se poursuivre en cas d'échec des hooks. Avec **Retry**, le hook va retenter son code plusieurs fois avant que le déploiement ne prenne le relais en cas de réussite. Avec **Ignore**, alors le résultat du hook n'aura pas d'importance dans l'évolution du déploiement.

---

### Configurer le hook de pre-déploiement

Pour ajouter un hook de déploiement à notre déploiement, la commande pourrait être, à titre d'exemple, `oc set deployment-hook dc/<name> --pre -c <name> -- /bin/echo "Bonjour du hook de pre-déploiement !"`. Cette commande, basée sur `oc set`, va ajouter un hook de déploiement. On spécifie via un flag le fait qu'il s'agit d'un hook de pré-déploiement, via le paramètre `-c` le nom du conteneur, et enfin le script a exécuter (ici un simple `echo`). On peut observe l'apparition d'un pod prévu pour le hook via la commande `oc get pods -w`. Pour observer ici le résultat de notre hook, on peut observer les évènements, par exemple via la commande `oc get events`.

---

### La stratégie Recreate

Lorsque notre application ne doit avoir qu'une seule version présente à la fois, il peut être nécessaire d'utiliser la stratégie de déploiement **Recreate**. Créant un downtime, cette stratégie va nous permettre l'accès à un hook qui va être déclenché durant la période de transition entre les deux déploiements. Il peut être utile de faire, durant cette période, des tâches telles qu'une migration de base de données.  

---

### Readiness / Liveness

Lorsque l'on a besoin de connaitre l'état de fonctionnement de nos pods dans un déploiement, on va passer par des sondes telles qu'une sonde de **Liveness**, qui va regarder si l'application est dans un état d'erreur ou non. Une autre sonde, celle doucement nommée **Readiness**, va de son côté observer si le pod est en état de recevoir un traffic. Ces sonde vont faire des requêtes HTTP et vérifient que le retour se trouve être un code 200. Il est possible aussi de faire s'éxecuter une commande personnalisée à un interval voulu de sorte à avoir plus de contrôle. La sonde TCP va demander une connexion de type TCP pour vérifier l'état du déploiement au lieu du protocole HTTP. Elle est moins souvent utilisée, cependant.

Pour configurer une sonde, on va utiliser par exemple la commande `oc set probe dc/<name> --liveness --open-tcp=8080`. On voit qu'il s'agit ici d'une sonde d'état de sanité de notre déploiement de par le flag `--liveness`. On va également observer que celle-ci va passer par le protocole TCP pour effectuer ses vérification, au port 8080.

---

### Le Scaling dans Openshift (HPA - Horizontal Pod Scaling)

Dans Openshift, la promesse est la capacité de réaliser un scaling de type horizontal de notre application. Contrairement au scaling vertical, qui va doubler par exemple les ressources dans notre cluster. Le scaling horizontal, de son côté, va chercher plutôt à augmenter le nombre de processus tournant en parallèle. En répartissant la RAM et la puissance processeur entre plusieurs pods, on va obtenir à la fin un cluster plus résistant à l'erreur que si l'on avait un seul pod doté d'une puissance de calcul monumentale mais causant l'inactivité de notre application en cas d'erreur. 

---

### Modifier le nombre de replicats de notre application

Le nombre de replicas défini dans le fichier YAML de notre déploiement va fixer le nombre de pod présent à l'état de lancement de notre déploiement. Il est cependant possible de modifier le nombre de pods via la commande `oc scale` tel que `oc scale dc/<nom> --replicas=XXX`. On peut observer le journal d'évènement de notre déploiement en bas de la sortie de la commande `oc describe dc/<name>`.

Le fonctionnement du HPA, sur le papier, va chercher à augmenter ou baisser le nombre de pods en fonction du traffic entrant de sorte à adapter la puissance de notre application en fonction de son utilisation et ainsi faire des heureux et / ou des économies. Pour ce faire, il va utiliser une opération mathémtique, une équation prenant en compte le nombre de pods, les ressources matérielles disponibles telles que la RAM ou le CPU ainsi que la valeur désirée de puissance de calcul. On obtient alors `Nb_Pods*Target_CPU*Requested_CPU_Per_Pod`.

Pour créer un autoscaler, on peut utiliser la commande `oc autoscale dc/<nom> --min XXX --max XXX --cpu-percent=XXX` de sorte à laisser à Openshift le loisir de faire les mathématiques et nous d'apprécier simplement le résultat. On peut ensuite observer notre ressource via `oc describe hpa/<name>`, ou en lire le fichier YAML via `oc get -o yaml hpa/<name>`.

---

### Les Templates

La dernière ressource Openshift que nous allons voir se nomme **Template**. Elles servent à regrouper en un fichier de configuration unique l'ensemble des ressources que l'on pourrait avoir besoin de déployer dans notre cluster pour rendre disponible notre application. Chaque template est composée de trois parties: 
- **Préambule**: Va contenir une sorte de header définissant les métadonnés et le type d'élément
- **Liste d'objet**: Une série d'objet YAML se trouvant dans une liste d'objets
- **Parametres**: Des identifiants, des chaines de connexions, cette section permet de rendre la template modifiable et personnalisable.

---

### Utiliser une template 

Pour utiliser une template, il est possible de modifier des fichiers YAML et de les envoyer à Openshift via la commande `oc create -f <chemin/vers/fichier>.yaml`. Pour vérifier l'arrivée du template sur notre serveur, il est possible de les lister via la commande `oc get template`. Si l'on veut l'utiliser, il suffit ensuite d'utiliser le nom de notre template dans la commande `oc new-app <template-name>`.

---

### Paramétrer les templates

Il est possible de paramétrer nos template via les fichiers YAML. La partie qui nous intéresse est la section `parameters` qui a pour but d'aider les utilisateurs à comprendre et se servir de notre template. Chaque paramètre du listing se compose d'une **description**, d'un **nom d'affichage** pour notre template, d'un **nom de variable** à peupler et enfin d'une **valeur par défaut** en cas de non peuplement du paramètre.

Pour fixer les paramètres de notre template lors du déploiement d'une application, on va utiliser le paramètre de commande `-p` tel que: `oc new-app <template-name> -p <VARIABLE>=<Valeur>`.

---

### La gestion des templates

Pour utiliser une template, on peut également afficher simplement la liste des objets résultats du template, pour par exemple pouvoir ensuite les analyser ou les déploiement un à un. Pour ce faire, on va utiliser la commande `oc process <nom-template>`. Cette commande va retourner un JSON, contrairement à la majorité des commandes d'Openshift. On peut cependant demander la sortie en YAML via l'ajout de `-o yaml`. Il est possible d'ajouter les paramètres à notre commande, de sorte à pouvoir observer la sortie selon ces paramètres (via `-p KEY=value`). 

En redirigeant le résultat de la commande `oc process`, on peut par exemple obtenir un fichier déplaçable ou partageable. On peut également l'éditer avec un éditeur de texte. Une fois un fichier YAML obtenu, on pourra l'utiliser à l'avenir, par exemple avec la commande `oc create -f <filepath>.yaml`