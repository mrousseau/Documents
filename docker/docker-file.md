# Docker-File

## exercices:

### ex1 

* 1 - Build image from ubuntu
* 2 - update repo (with apt)
* 3 - install curl, nodejs, build-essential
* 4 - copy work file
* 5 - run node js to execute helloworld

```
FROM ubuntu:latest

RUN apt-get update --yes

RUN apt-get install --yes curl

RUN curl --silent  --location http://deb/nodesource.com/setup_14.x

RUN apt-get install --yes nodejs

RUN apt-get install --yes build-essential

COPY ./helloworld.js /var/www§

CMD ["node", "/var/wwww/helloworld.js" ]

```

```
console.log("Hello World");                 # helloworld.js
```


### docker commands

```
 docker build -t nodejs:latest             # construire une image docker avec le nomage ( -t <le_nom>)
```