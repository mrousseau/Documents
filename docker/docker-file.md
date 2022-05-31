# Dockerfile

## exercices:

### ex1 

* 1 - Build image from ubuntu
* 2 - update repo (with apt)
* 3 - install curl, nodejs, build-essential
* 4 - copy work file
* 5 - run node js to execute helloworld

```
FROM ubuntu:latest
# commentaire : liste des Runs
RUN apt-get update --yes

RUN apt-get install --yes curl

RUN curl --silent  --location https://deb.nodesource.com/setup_18.x

RUN apt-get install --yes nodejs

RUN apt-get install --yes build-essential

COPY ./helloworld.js /var/www/

CMD ["node", "/var/www/helloworld.js" ]

```

##V2
```
FROM ubuntu:latest
# commentaire : liste des Runs
RUN apt-get update --yes

RUN apt-get install --yes curl

RUN curl --silent  --location https://deb.nodesource.com/setup_18.x

RUN apt-get install --yes nodejs

RUN apt-get install --yes build-essential

WORKDIR /var/www/

COPY ./helloworld.js .

CMD ["node", "helloworld.js" ]
```

```
console.log("Hello World");                 # helloworld.js 
```

```
ADD # argument récupérer des fichier des éléments d'une source exterieur décompresse automatiquement
WORKDIR # définit l'emplacement de travail dans le container
ENTRYPOINT # fait la mmême chose que CMD mais ne permet pas d'insertion exemple docker run  nodejs:test echo "blabla" ne fonctionnera pas donc permet d'éviter a l'utilisateur de rajouter des instruction lors de l'execution.

ARG #permet de passer des params pour le build
```



### docker commands

```
 docker build -t nodejs:latest .             # construire une image docker avec le nomage ( -t <le_nom>),  (.) pour indiquer l'emplacemende des fichiers pour l'image
```