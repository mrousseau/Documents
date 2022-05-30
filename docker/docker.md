# Docker

```
docker run <nom_de_l_image>                                             #démarrer un container
```


```
docker run -it <nom_de_l_image> bash                                    #executer dans le container
            i -> interactive
            t -> terminal 
```

quand on lance docker run on lance docker create puis docker container start

## gérer des container 


docker container
```
docker image ls                                                         # lister les images dl sur le poste
```
```
docker image ls -q                                                      # lister les images dl sur le poste mais avoir que les ids
```
```
docker image rm  <id de l image>                                        # permet de supprimer l'image sur ton poste
```
```
docker conatiner ls                                                     # permet de lister les images docker actifs
```
```
docker container ls -a                                                  # permet de lister les conatiners sur la machine
```
```
docker ps                                                               # permet de lister les conatiners sur la machine
```
```
docker ps -a
```
```
docker container prune                                                  # supprimer tout les container de la machine
```
```
docker container attach <id du container>                               # permet de s'attacher a la cmd du container
```
```
docker container start -ai <id du container>                            # permet de démarrer un container avec son id puis de s'attacher
```
```
docker container restart <id du container>                              # permet de redémarrer mon container
```
```
docker container stop <id du container>                                 # permet de stopper le container
```
```
docker container kill <id du container>                                 # permet de stopper le container de façon brutal
```

## bonus : 
```
docker container rename <id du container par id ou nom> <nouveau nom>   # renommer des containers
```
```
docker container pause <nom du container ou id>                         # met le container en pause
```
```
docker conatiner exec <nom du container ou id> <exec ma cmd>            # executer une cmd dans le container
        ex: docker conatiner exec myContaier touche /home/fichier.txt  
```

```
docker container cp ./file.txt myContainer:/home                        # copie un fichier dans un container
```
