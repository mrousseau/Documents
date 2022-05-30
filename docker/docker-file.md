# Docker-File

## exercices:

### ex1 

* 1 - Build image from ubuntu
* 2 - update repo (with apt)
* 3 - install curl, nodejs, build-essential
* 4 - copy work file
* 5 - run node js to execute helloworld

```
FROM ubuntu:latest                                                          # depuis l image d'ubuntu (derniere version)

RUN apt-get update --yes                                                    # un layer pour mettre a jour les repo

RUN apt-get install --yes curl                                              # un layer pour installer curl

RUN curl --silent  --location http://deb/nodesource.com/setup_14.x          # un layer pour dl lle repo de node js 

RUN apt-get install --yes nodejs                                            # un layer pour installer node js

RUN apt-get install --yes build-essential                                   # un layer pour insaller les build-essential 

COPY ./helloworld.js /var/www/                                              # on copie le fichier à executer dans l'emplacement qui se trouve dans le container dans l'exemple \var\www\

CMD ["node", "/var/wwww/helloworld.js" ]                                    # on lance le script js 

```

```
console.log("Hello World");                 # helloworld.js 
```


### docker commands

```
 docker build -t nodejs:latest .             # construire une image docker avec le nomage ( -t <le_nom>)
```