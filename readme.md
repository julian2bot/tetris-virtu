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

### QUESTION 3 - Le démarrage des services peut être compromis si d’autres processus de la machine virtuelle sont déjà en écoute sur certains ports. Quels sont le ou les ports concernés?  

les ports concernés sont : 8081
et 8080 / 6379 mais perso pas eu de probleme avec.



