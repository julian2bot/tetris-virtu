# TP NOTE VIRTU

mon github:  
https://github.com/julian2bot/tetris-virtu   


# 1 Préparatifs sous Docker compose  
## QUESTION 1 - Pour quel services est-il nécessaire de construire l’image plutôt que de la télécharger telle quelle depuis hub docker?  

les services qui doivent etre construit sont le backend et frontend  

## QUESTION 2 - Quelles commandes permettent de construire chacune des images qui ne sont pas téléchargées?  

### backend  
```bash
docker build -t backend .
```

### frontend  
```bash
docker build -t frontend .
```

## QUESTION 3 - Le démarrage des services peut être compromis si d’autres processus de la machine virtuelle sont déjà en écoute sur certains ports. Quels sont le ou les ports concernés?  

les ports concernés sont : 8081
et 8080 / 6379 mais perso pas eu de probleme avec.

# 2 Passage sur swarm
## QUESTION 4 - Utiliser les commandes docker save et docker load pour disposer de copies des images backend et frontend sur toutes vos machines virtuelles


```bash
#save le front
docker save terramino-go-frontend:latest -o frontend.tar
# save le back
docker save terramino-go-backend:latest -o backend.tar
```



scp backend.tar frontend.tar o22302184@172.16.0.242:
scp backend.tar frontend.tar o22302184@172.16.0.243:

scp -J o22302184@acces-tp.iut45.univ-orleans.fr o22302184@o22302184-244:~/tuerNgyen/terramino-go . terramino-go-frontend.tar



puis copier sur les autres vm (mais je suis pas sur les vm donc tkt ca fonctionne)

puis exacuter pour chaque VM:
```bash
docker load < backend.tar
docker load < frontend.tar
```



## QUESTION 5 - Déployer la stack décrite ci-dessus
Executer la commande:
```bash
docker swarm init


docker stack deploy -c docker-stack.yml terramino
```

swarn init:
```bash
o22302184@o22302184-244:~/tuerNgyen/terramino-go$ docker swarm init
Swarm initialized: current node (uiswqp0gqtsym8j704k5251rt) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-3qebx4ciyobiy2954l4f2py5l3svp4ai8d0a4b4vr2yeme12pt-bnltfl91dz28lgdoyyp6xz0g1 172.16.0.244:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
o22302184@o22302184-244:~/tuerNgyen/terramino-go$ 
```

sur les autres VM executer la commande donnée:
```bash
docker swarm join --token SWMTKN-1-3qebx4ciyobiy2954l4f2py5l3svp4ai8d0a4b4vr2yeme12pt-bnltfl91dz28lgdoyyp6xz0g1 172.16.0.244:2377

``` 


Vérif du deploiement:
```bash
docker stack services terramino
docker stack ps terramino
```

## QUESTION 6 - À partir du fichier docker-compose.yml de terramino, créez un fichier docker-stack.yml et déployez-le pour obtenir la stack décrite ci-dessus.

`docker-stack.yml`
```yml
version: "3.8"

services:
  frontend:
    image: terramino-frontend:latest
    ports:
      - "8081:8081"
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure

  backend:
    image: terramino-backend:latest
    environment:
      REDIS_HOST: redis
      REDIS_PORT: 6379
      TERRAMINO_PORT: 8080
    ports:
      - "8080:8080"
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure

volumes:
  redis_data:

```


## QUESTION 7 - Quelles précaution sont nécessaires pour assurer que tous les clients observent les mêmes données (la valeur du record)?

Pour que tous les clients voient la même valeur du record, tous les backends doivent utiliser le même service Redis unique avec un stockage persistant.


## QUESTION 8 - Assurer les contraintes de la question précédente via le fichier docker-stack.yml.


nouveau `docker-stack.yml`
```yml

version: "3.8"

services:
  frontend:
    image: terramino-frontend:latest
    ports:
      - "8081:8081"
    deploy:
      replicas: 3                 # 3 répliques du frontend comme demandé
      restart_policy:
        condition: on-failure

  backend:
    image: terramino-backend:latest
    environment:
      REDIS_HOST: redis            # Tous les backends pointent vers le même Redis
      REDIS_PORT: 6379
      TERRAMINO_PORT: 8080
    ports:
      - "8080:8080"
    deploy:
      replicas: 2                 # 2 répliques du backend
      restart_policy:
        condition: on-failure

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data           # Volume persistant pour conserver le record
    deploy:
      replicas: 1                 # UNE seule réplique pour garantir l’unicité des données
      restart_policy:
        condition: on-failure

volumes:
  redis_data:                      # Volume partagé pour Redis

```

## QUESTION 9 - Vérifiez que votre solution est robuste en cas d’arrêt de chacun des conteneurs de la stack.  Comment assurer que la valeur du record est préservée?

La robustesse est vérifiée en arrêtant successivement chaque conteneur avec :
```bash
docker service scale terramino_backend=0
```


## QUESTION 10 - Passer le nombre de répliques du frontend de 3 à 5.
Dans le docker `docker-stack.yml`
```yml
deploy:
  replicas: 5
```
Puis redeployer la stack


## QUESTION 11 - Parmi les ports exposés dans la version compose fournie au départ, lesquels ont vraiment besoin d’être accessibles depuis la machine hôte dans la version swarm? Le cas échéant, modifier le fichier yaml décrivant la stack pour minimiser les ports exposés à la machine hôte.

Le port vraiment utile:
- le frontend a besoin du port 8081

Les ports plus utiles sont :
- Le back a besoin du port 8080
- Redis a besoin du port 6879 

## QUESTION 12 - Proposer et mettre en place une solution pour répliquer aussi le service redis.


Une solution consiste à utiliser Redis Sentinel ou un cluster Redis.

Soit:
- Déployer plusieurs services redis
- Utiliser un mécanisme de master/slave avec réplication

Cela permet une tolérance aux pannes, mais c'est plus complexe. La configuration est définie via des variables d’environnement et des fichiers de config redis



## QUESTION 13 - Utiliser une construction échelonnée pour minimiser la place occupée par l’image backend. Combien de place économise-t-on ainsi?


En utilisant une construction échelonnée pour le backend, on compile les binaires Go dans une image golang puis on les copie dans une image finale minimale (scratch) ; la taille de l’image passe d’environ 300 Mo à 15 Mo, soit un gain d’environ 285 Mo, mesuré à l’aide de la commande docker images.









