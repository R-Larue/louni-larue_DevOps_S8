# DevOps Larue-Louni S8

## Docker TP

### Partie 1 - 30/01 'Discover Docker'

#### Database

##### Identifiants

- POSTGRES_DB=db
- POSTGRES_USER=usr
- POSTGRES_PASSWORD=pwd

##### Commandes

```
docker build .
docker run --name postgres -p 3306:5432 alpine-14.1
docker start postgres
```

Ainsi on se connecte à la base de données 'db' avec les identifiants fournis.

#### Adminer
##### Network

```
docker network create app-network
```
##### Redemarré Postgres & Lancer Adminer

```
docker stop postgres
docker rm postgres
docker run --name postgres --network app-network -p 5432:5432 -d alpine-14.1
docker run -p "8080:8080" --net=app-network --name=adminer -d adminer
```

Lorsqu'on supprime le conteneur postgres, les données en base de données ne sont plus persistentes.

##### Rendre les données persistentes

```
docker run --name postgres --network app-network -p 5432:5432 -v /home/mutton/DevOps/TP1/data:/var/lib/postgresql/data -d alpine-14.1
```

#### BAckend API