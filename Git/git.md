# Git Command

## Commande de Base

### Initialise le projet GIT
```
git init   
```

### Ajoute un fichier au commit 
```
git add <fileName>
```

### Ajoute tous les fichiers modifiés dans le répertoire
```
git add .
```

### Permet de connaitre le status du fichier modifié / ajouté etc... 
```
git status
```

### Permet de faire un commit en local c'est à dire de pousser le code uniquement sur son serveur en local de son poste
```
git commit -m "mon message"
```

### Permet de faire un push sur le serveur distant et envoie tous les commits fait en amont
```
git push
```


### Rattacher un projet initialisé a un dépôt distant (pré requis le projet est déjà initialisé et les fichiers est déjà ajouté et commit )
```
git remote add origin <url_du_repository>
git branch -M main
git push -uf origin main
```





