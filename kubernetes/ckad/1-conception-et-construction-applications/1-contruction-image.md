# Processus de construction d'une image de conteneur avec docker

Nous allons proceder à la creation de l'image d'une application web affichand le message *Welcome to my page !*<br>

## Construction de l'image

Nous allons suivre les etapes ci-dessous, dans un terminal windows ou linux executer les commandes ci-dessous: 
NB: notre cas nous le faisons depuis un terminal linux

- Nous créons le répertoire de travail
```
mkdir website
cd website
``` 

- Nous créons un fichier *index.html* avec le contenur du message
```
vi index.html
```

```
Welcome to my page !
```

- Nous créons notre fichier *Dockerfile*
```
vi Dockerfile
```

```
FROM nginx:stable
COPY index.html /usr/share/nginx/html/
```

**nginx:stable** est notre image de base de base 

**COPY index.html /usr/share/nginx/html/** permet la copie du fichier index dans le repertoire /usr/share/nginx/html/ de notre image de base **nginx:stable**  

- Nous construisons notre image version 0.0.1 avec la commande docker, nous pouvons aussi utiliser les commandes podman ou crictl
```
docker build -t website:1.0.0 .
```

## Exploitation du container de l'image
- Nous créons un conteneur à partir de l'image *website:1.0.0* créé ci-dessus

```
docker run --rm --name website -d -p 8080:80 website:1.0.0
```

- Nous pouvons vérifier si le conteneur écoute bien sur le port 8080
```
curl localhost:8080
```

- Nous pouvons stopper et supprimer notre conteneur
```
docker container stop website
```

```
docker container rm website
```

- Nous pouvons sauvegarder notre image dans un fichier archivé
```
docker save -o $HOME/website.1.0.0.tar website:1.0.0
```

L'option *-o* permet d'indiquer la destination de la sauvegarde.<br>

- Nous pouvons aussi vérifier si le fichier de sauvegarde a été créé
```
ls $HOME
```
*Le tutoriel ci-dessous s'inspire des cours CKAD sur ACLOUD GURU et la plateforme GITHUB willbrid de **WILLIAM NGASSAM***

