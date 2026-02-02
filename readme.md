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



scp backend.tar frontend.tar o22302184@172.16.0.244:/home/user/


scp -J o22302184@acces-tp.iut45.univ-orleans.fr o22302184@o22302184-244:~/tuerNgyen/terramino-go . terramino-go-frontend.tar





