# Concepts de déploiement

Lors du déploiement d'une application **FastAPI**, ou en fait, de n'importe quel type d'API web, il y a plusieurs concepts qui vous intéressent probablement, et en les utilisant vous pouvez trouver la **manière la plus appropriée** de **déployer votre application**.

Voici quelques-uns des concepts importants :

* Sécurité - HTTPS
* Exécution au démarrage
* Redémarrage
* Réplication (nombre de processus en cours d'exécution)
* Mémoire
* Étapes précédentes avant le démarrage

Nous verrons comment ils affectent les **déploiements**.

En fin de compte, l'objectif ultime est de pouvoir **servir vos clients API** d'une manière **sécurisée**, d'**éviter les perturbations** et d'utiliser les **ressources informatiques** (par exemple les serveurs distants/machines virtuelles) aussi efficacement que possible. 🚀

Je vais vous en dire un peu plus sur ces **concepts** ici, et j'espère que cela vous donnera l'**intuition** dont vous aurez besoin pour décider comment déployer votre API dans des environnements très différents, peut-être même dans des **futurs** qui n'existent pas encore.

En tenant compte de ces concepts, vous pourrez **évaluer et concevoir** la meilleure façon de déployer **vos propres API**.

Dans les prochains chapitres, je vous donnerai plus de **recettes concrètes** pour déployer des applications FastAPI.

Mais pour l'instant, vérifions ces **idées conceptuelles** importantes. Ces concepts s'appliquent également à tout autre type d'API web. 💡

## Sécurité - HTTPS

Dans le [chapitre précédent sur le HTTPS](./https.md){.internal-link target=_blank}, nous avons appris comment le HTTPS assure le cryptage de votre API.

Nous avons également vu que HTTPS est normalement fourni par un composant **externe** à votre serveur d'application, un **TLS Termination Proxy**.

Et il doit y avoir quelque chose en charge du **renouvellement des certificats HTTPS**, cela peut être le même composant ou quelque chose de différent.

### Exemples d'outils pour HTTPS

Voici quelques-uns des outils que vous pouvez utiliser en tant que proxy de terminaison TLS :

* Traefik
    * Traefik gère automatiquement les renouvellements de certificats ✨
* Caddy
    Gère automatiquement les renouvellements de certificats ✨ * Caddy
* Nginx
    * Avec un composant externe comme Certbot pour les renouvellements de certificats
* HAProxy
    * Avec un composant externe comme Certbot pour le renouvellement des certificats
* Kubernetes avec un contrôleur Ingress comme Nginx
    * Avec un composant externe comme cert-manager pour le renouvellement des certificats.
* Géré en interne par un fournisseur de cloud dans le cadre de ses services (voir ci-dessous 👇)

Une autre option consiste à utiliser un **service en nuage** qui effectue une plus grande partie du travail, y compris la mise en place de HTTPS. Il pourrait avoir certaines restrictions ou vous faire payer plus cher, etc. Mais dans ce cas, vous n'auriez pas à configurer vous-même un proxy de terminaison TLS.

Je vous montrerai des exemples concrets dans les prochains chapitres.

---

Les concepts suivants à prendre en compte concernent le programme qui exécute votre API (par exemple Uvicorn).

## Programme et processus

Nous parlerons beaucoup du "**processus**", il est donc utile de clarifier ce qu'il signifie, et quelle est la différence avec le mot "**programme**".

### Qu'est-ce qu'un programme ?

Le mot **programme** est couramment utilisé pour décrire de nombreuses choses :

* Le **code** que vous écrivez, les **fichiers Python**.
* Le **fichier** qui peut être **exécuté** par le système d'exploitation, par exemple : `python`, `python.exe` ou `uvicorn`.
* Un programme particulier lorsqu'il **s'exécute** sur le système d'exploitation, utilise le processeur et stocke des données dans la mémoire. On l'appelle aussi **processus**.

### Qu'est-ce qu'un processus ?

Le mot **processus** est normalement utilisé de manière plus spécifique, en se référant uniquement à ce qui s'exécute dans le système d'exploitation (comme dans le dernier point ci-dessus) :

* Un programme particulier pendant qu'il **s'exécute** sur le système d'exploitation.
    * Cela ne fait pas référence au fichier, ni au code, mais **spécifiquement** à la chose qui est **exécutée** et gérée par le système d'exploitation.
* Tout programme, tout code, **ne peut faire des choses** que lorsqu'il est **exécuté**. Ainsi, lorsqu'un **processus est en cours d'exécution**.
* Le processus peut être **terminé** (ou "tué") par vous ou par le système d'exploitation. À ce moment-là, il cesse de fonctionner ou d'être exécuté, et il ne peut **plus faire de choses**.
* Chaque application en cours d'exécution sur votre ordinateur a un processus derrière elle, chaque programme en cours d'exécution, chaque fenêtre, etc. Et il y a normalement de nombreux processus en cours d'exécution **en même temps** lorsqu'un ordinateur est allumé.
* Il peut y avoir **plusieurs processus** du **même programme** en cours d'exécution en même temps.

Si vous consultez le "gestionnaire de tâches" ou le "moniteur système" (ou des outils similaires) de votre système d'exploitation, vous pourrez voir un grand nombre de ces processus en cours d'exécution.

Par exemple, vous verrez probablement que plusieurs processus exécutent le même programme de navigation (Firefox, Chrome, Edge, etc.). Ils exécutent normalement un processus par onglet, plus quelques autres processus supplémentaires.

<img class="shadow" src="../../../en/docs/img/deployment/concepts/image01.png">

---

Maintenant que nous connaissons la différence entre les termes **processus** et **programme**, continuons à parler des déploiements.

## Exécution au démarrage

Dans la plupart des cas, lorsque vous créez une API web, vous souhaitez qu'elle soit **toujours en cours d'exécution**, sans interruption, afin que vos clients puissent toujours y accéder. Bien sûr, sauf si vous avez une raison spécifique pour laquelle vous voulez qu'elle ne fonctionne que dans certaines situations, mais la plupart du temps, vous voulez qu'elle fonctionne en permanence et qu'elle soit **disponible**.

### Dans un serveur distant

Lorsque vous configurez un serveur distant (un serveur cloud, une machine virtuelle, etc.), la chose la plus simple que vous puissiez faire est de lancer Uvicorn (ou similaire) manuellement, de la même manière que vous le faites lorsque vous développez localement.

Et cela fonctionnera et sera utile **pendant le développement**.

Mais si votre connexion au serveur est perdue, le **processus d'exécution** mourra probablement.

Et si le serveur est redémarré (par exemple après des mises à jour ou des migrations depuis le fournisseur de cloud), vous ne le remarquerez probablement **pas**. Et pour cette raison, vous ne saurez même pas que vous devez redémarrer le processus manuellement. Ainsi, votre API restera simplement morte. 😱

### Exécuter automatiquement au démarrage

En général, vous voudrez probablement que le programme serveur (par exemple Uvicorn) soit lancé automatiquement au démarrage du serveur, et sans avoir besoin d'une **intervention humaine**, pour avoir un processus toujours en cours d'exécution avec votre API (par exemple Uvicorn exécutant votre application FastAPI).

### Programme séparé

Pour y parvenir, vous aurez normalement un **programme séparé** qui s'assurera que votre application est exécutée au démarrage. Et dans de nombreux cas, il s'assurera également que d'autres composants ou applications sont également lancés, par exemple, une base de données.

### Exemples d'outils à exécuter au démarrage

Voici quelques exemples d'outils qui peuvent faire ce travail :

* Docker
* Kubernetes
* Docker Compose
* Docker en mode Swarm
* Systemd
* Superviseur
* Géré en interne par un fournisseur de services en nuage dans le cadre de ses services
* D'autres encore...

Je vous donnerai des exemples plus concrets dans les prochains chapitres.

## Redémarrages

Tout comme vous voulez vous assurer que votre application est lancée au démarrage, vous voulez probablement aussi vous assurer qu'elle est **redémarrée** après des échecs.

### Nous faisons des erreurs

En tant qu'humains, nous faisons des **erreurs**, tout le temps. Les logiciels ont presque *toujours* des **bugs** cachés à différents endroits. 🐛

Et nous, en tant que développeurs, continuons à améliorer le code au fur et à mesure que nous trouvons ces bogues et que nous implémentons de nouvelles fonctionnalités (en ajoutant éventuellement de nouveaux bogues aussi 😅).

### Les petites erreurs sont automatiquement gérées

Lors de la création d'API web avec FastAPI, s'il y a une erreur dans notre code, FastAPI la contiendra normalement à la requête unique qui a déclenché l'erreur. 🛡

Le client recevra une **500 Internal Server Error** pour cette requête, mais l'application continuera à fonctionner pour les requêtes suivantes au lieu de se planter complètement.

### Les plus grosses erreurs - les plantages

Néanmoins, il peut y avoir des cas où nous écrivons du code qui **crash l'application entière** faisant planter Uvicorn et Python. 💥

Et pourtant, vous ne voudriez probablement pas que l'application reste morte parce qu'il y a eu une erreur à un endroit, vous voudriez probablement qu'elle **continue à fonctionner** au moins pour les *opérations de parcours* qui ne sont pas cassées.

### Redémarrer après un crash

Mais dans les cas d'erreurs vraiment graves qui font planter le **processus** en cours d'exécution, vous voudriez un composant externe qui soit chargé de **redémarrer** le processus, au moins deux ou trois fois...

!!! astuce
    ...Bien que si l'application entière **se plante immédiatement**, cela n'a probablement pas de sens de la redémarrer sans cesse. Mais dans ce cas, vous le remarquerez probablement pendant le développement, ou au moins juste après le déploiement.

    Concentrons-nous donc sur les cas principaux, où le système pourrait se bloquer entièrement dans certains cas particuliers **dans le futur**, et où il est toujours utile de le redémarrer.

Vous voudriez probablement que la chose chargée de redémarrer votre application soit un **composant externe**, parce qu'à ce moment-là, la même application avec Uvicorn et Python a déjà planté, donc il n'y a rien dans le même code de la même application qui puisse faire quoi que ce soit à ce sujet.

### Exemples d'outils pour redémarrer automatiquement

Dans la plupart des cas, le même outil qui est utilisé pour **exécuter le programme au démarrage** est également utilisé pour gérer les **redémarrages** automatiques.

Par exemple, cela peut être géré par :

* Docker
* Kubernetes
* Docker Compose
* Docker en mode Swarm
* Systemd
* Superviseur
* Géré en interne par un fournisseur de services en nuage dans le cadre de ses services
* Autres...

## Réplication - Processus et mémoire

Avec une application FastAPI, utilisant un programme serveur comme Uvicorn, l'exécuter une fois dans **un processus** peut servir plusieurs clients simultanément.

Mais dans de nombreux cas, vous voudrez exécuter plusieurs processus de travail en même temps.

### Processus multiples - Travailleurs

Si vous avez plus de clients que ce qu'un seul processus peut gérer (par exemple, si la machine virtuelle n'est pas trop grande) et que vous disposez de **multiples cœurs** dans le processeur du serveur, vous pouvez alors avoir **multiples processus** fonctionnant avec la même application en même temps, et distribuer toutes les demandes entre eux.

Lorsque vous exécutez **plusieurs processus** du même programme API, ils sont communément appelés **workers**.

### Processus et ports de travailleur

Vous vous souvenez de la documentation [À propos de HTTPS](./https.md){.internal-link target=_blank} selon laquelle un seul processus peut être à l'écoute sur une combinaison de port et d'adresse IP dans un serveur ?

C'est toujours le cas.

Ainsi, pour pouvoir avoir **plusieurs processus** en même temps, il doit y avoir **un seul processus à l'écoute sur un port** qui transmet ensuite la communication à chaque processus travailleur d'une manière ou d'une autre.

### Mémoire par processus

Lorsque le programme charge des éléments en mémoire, par exemple un modèle d'apprentissage automatique dans une variable ou le contenu d'un gros fichier dans une variable, tout cela **consomme un peu de la mémoire (RAM)** du serveur.

Et les processus multiples ne **partagent normalement pas de mémoire**. Cela signifie que chaque processus en cours d'exécution a ses propres éléments, variables et mémoire. Et si vous consommez une grande quantité de mémoire dans votre code, **chaque processus** consommera une quantité équivalente de mémoire.

### Mémoire du serveur

Par exemple, si votre code charge un modèle d'apprentissage automatique d'une taille de **1 Go**, lorsque vous exécutez un processus avec votre API, il consommera au moins 1 Go de RAM. Et si vous lancez **4 processus** (4 travailleurs), chacun consommera 1 Go de RAM. Au total, votre API consommera donc **4 Go de RAM**.

Et si votre serveur distant ou votre machine virtuelle ne dispose que de 3 Go de RAM, essayer de charger plus de 4 Go de RAM posera des problèmes. 🚨

### Processus multiples - Un exemple

Dans cet exemple, il y a un **processus gestionnaire** qui démarre et contrôle deux **processus travailleurs**.

Ce processus gestionnaire est probablement celui qui écoute sur le **port** de l'IP. Il transmet toutes les communications aux processus de travail.

Ces processus travailleurs seraient ceux qui exécutent votre application, ils effectueraient les principaux calculs pour recevoir une **requête** et renvoyer une **réponse**, et ils chargeraient tout ce que vous mettez dans les variables en RAM.

<img src="../../../en/docs/img/deployment/concepts/process-ram.svg">

Et bien sûr, sur la même machine, d'autres **processus** sont probablement en cours d'exécution, en dehors de votre application.

Un détail intéressant est que le pourcentage de **CPU utilisé** par chaque processus peut **varier** considérablement dans le temps, mais la **mémoire (RAM)** reste normalement plus ou moins **stable**.

Si vous avez une API qui effectue une quantité comparable de calculs à chaque fois et que vous avez beaucoup de clients, l'**utilisation de l'UC** sera probablement *également stable* (au lieu d'augmenter et de diminuer rapidement).

### Exemples d'outils et de stratégies de réplication

Il peut y avoir plusieurs approches pour y parvenir, et je vous en dirai plus sur des stratégies spécifiques dans les prochains chapitres, par exemple lorsque je parlerai de Docker et des conteneurs.

La principale contrainte à prendre en compte est qu'il doit y avoir un **seul** composant gérant le **port** dans l'**IP publique**. Il doit ensuite avoir un moyen de **transmettre** la communication aux **processus/travailleurs** répliqués.

Voici quelques combinaisons et stratégies possibles :

* **Gunicorn** gérant des **travailleurs Uvicorn**
    * Gunicorn serait le **gestionnaire de processus** écoutant sur l'**IP** et le **port**, la réplication se ferait en ayant **plusieurs processus de travailleur Uvicorn**.
**Uvicorn** gère **les travailleurs Uvicorn**.
    * Un **gestionnaire de processus** Uvicorn écouterait sur **IP** et **port**, et démarrerait **plusieurs processus de travailleur Uvicorn**.
* **Kubernetes** et autres **systèmes de conteneurs distribués**
    * Quelque chose dans la couche **Kubernetes** écouterait sur **IP** et **port**. La réplication se ferait en ayant **plusieurs conteneurs**, chacun avec **un processus Uvicorn** en cours d'exécution.
* Les **services en nuage** qui s'en chargent pour vous
    * Le service en nuage va probablement **gérer la réplication pour vous**. Il vous permettra peut-être de définir **un processus à exécuter**, ou une **image de conteneur** à utiliser, dans tous les cas, il s'agira très probablement **d'un seul processus Uvicorn**, et le service cloud se chargera de le répliquer.

!!! conseil
    Ne vous inquiétez pas si certains de ces éléments concernant les **conteneurs**, Docker ou Kubernetes n'ont pas encore beaucoup de sens.

    Je vous en dirai plus sur les images de conteneurs, Docker, Kubernetes, etc. dans un prochain chapitre : [Déploiement - Déployer avec Docker](./docker.md){.internal-link target=_blank}.
## Étapes précédentes avant le démarrage

Dans de nombreux cas, vous souhaitez effectuer certaines étapes **avant de démarrer** votre application.

Par exemple, vous pourriez vouloir exécuter des **migrations de base de données**.

Mais dans la plupart des cas, vous ne voudrez exécuter ces étapes qu'une seule fois.

Vous voudrez donc disposer d'un **processus unique** pour exécuter ces **étapes précédentes**, avant de démarrer l'application.

Et vous devrez vous assurer que c'est un seul processus qui exécute ces étapes précédentes *même* si par la suite, vous démarrez **plusieurs processus** (plusieurs travailleurs) pour l'application elle-même. Si ces étapes étaient exécutées par **plusieurs processus**, ils **dupliqueraient** le travail en l'exécutant en **parallèle**, et si les étapes étaient quelque chose de délicat comme une migration de base de données, elles pourraient causer des conflits les unes avec les autres.

Bien sûr, il y a des cas où il n'y a pas de problème à exécuter les étapes précédentes plusieurs fois, dans ce cas, c'est beaucoup plus facile à gérer.

!!! conseil
    Gardez également à l'esprit que, selon votre configuration, dans certains cas, vous **n'aurez même pas besoin d'étapes précédentes** avant de démarrer votre application.

    Dans ce cas, vous n'aurez pas à vous soucier de tout cela. 🤷

### Exemples de stratégies pour les étapes précédentes

Cela **dépendra fortement** de la façon dont vous **déployez votre système**, et sera probablement lié à la façon dont vous démarrez les programmes, en gérant les redémarrages, etc.

Voici quelques idées possibles :

* Un "Init Container" dans Kubernetes qui s'exécute avant votre conteneur d'application.
* Un script bash qui exécute les étapes précédentes et démarre ensuite votre application.
    * Vous auriez toujours besoin d'un moyen de démarrer/redémarrer *ce* script bash, de détecter les erreurs, etc.

!!! astuce
    Je vous donnerai des exemples plus concrets pour faire cela avec des conteneurs dans un prochain chapitre : [Déploiement - Déployer avec Docker](./docker.md){.internal-link target=_blank}.

## Utilisation des ressources

Votre (vos) serveur(s) est (sont) une **ressource**, vous pouvez consommer ou **utiliser**, avec vos programmes, le temps de calcul sur les CPU, et la mémoire RAM disponible.

Quelle part des ressources du système souhaitez-vous consommer/utiliser ? Il peut être facile de penser "pas beaucoup", mais en réalité, vous voudrez probablement consommer **le plus possible sans planter**.

Si vous payez pour 3 serveurs mais que vous n'utilisez qu'une petite partie de leur RAM et de leur CPU, vous êtes probablement en train de **gaspiller de l'argent** 💸, et probablement de **gaspiller l'énergie électrique du serveur** 🌎, etc.

Dans ce cas, il serait préférable de n'avoir que deux serveurs et d'utiliser un pourcentage plus élevé de leurs ressources (CPU, mémoire, disque, bande passante réseau, etc.).

D'un autre côté, si vous avez 2 serveurs et que vous utilisez **100% de leur CPU et de leur RAM**, à un moment donné, un processus demandera plus de mémoire, et le serveur devra utiliser le disque comme "mémoire" (ce qui peut être des milliers de fois plus lent), ou même **crash**. Ou bien un processus peut avoir besoin d'effectuer un calcul et devra attendre que l'unité centrale soit à nouveau libre.

Dans ce cas, il serait préférable d'avoir **un serveur supplémentaire** et d'y exécuter quelques processus afin qu'ils aient **suffisamment de RAM et de temps CPU**.

Il est également possible que, pour une raison ou une autre, votre API connaisse une **poussée** d'utilisation. Elle est peut-être devenue virale, ou d'autres services ou bots commencent à l'utiliser. Vous voudrez peut-être disposer de ressources supplémentaires pour vous protéger dans ces cas-là.

Vous pouvez fixer un **nombre arbitraire** à cibler, par exemple, quelque chose **entre 50 % et 90 %** de l'utilisation des ressources. Le fait est que ce sont probablement les choses principales que vous voudrez mesurer et utiliser pour ajuster vos déploiements.

Vous pouvez utiliser des outils simples comme `htop` pour voir le CPU et la RAM utilisés sur votre serveur ou la quantité utilisée par chaque processus. Ou vous pouvez utiliser des outils de surveillance plus complexes, qui peuvent être distribués à travers les serveurs, etc.

## Récapitulation

Vous avez lu ici quelques-uns des principaux concepts que vous devrez probablement avoir à l'esprit lorsque vous déciderez de la manière de déployer votre application :

* Sécurité - HTTPS
* Exécution au démarrage
* Redémarrage
* Réplication (nombre de processus en cours d'exécution)
* Mémoire
* Étapes précédentes avant le démarrage

Comprendre ces idées et comment les appliquer devrait vous donner l'intuition nécessaire pour prendre toute décision lors de la configuration et de l'ajustement de vos déploiements. 🤓

Dans les prochaines sections, je vous donnerai des exemples plus concrets de stratégies possibles que vous pouvez suivre. 🚀
