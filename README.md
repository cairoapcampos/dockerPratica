# dockerPratica
Repositório com exemplos de docker e docker-compose

## docker run

**Atividade 1:**
 
* Crie os volumes: 

```
docker volume create wikijsdb_data
docker volume create wikijs_config
```

* Crie a rede utilizada:  `docker network create network-wikijs`

```
docker run -d --name wikijsdb \
 --hostname wikijsdb --restart always \
 --volume wikijsdb_data:/var/lib/postgresql/data \
 --network network-wikijs \
 -e "POSTGRES_DB=wikidb" \
 -e "POSTGRES_PASSWORD=wikipass" \
 -e "POSTGRES_USER=wikiuser" \
postgres:11-bullseye
```
```
docker run -d -p 8080:3000 \
--name wikijs --hostname wikijs \
--volume wikijs_config:/wiki/data/content \
--restart always \
--network network-wikijs \
-e "DB_TYPE=postgres" \
-e "DB_HOST=wikijsdb" \
-e "DB_PORT=5432" \
-e "DB_USER=wikiuser" \
-e "DB_PASS=wikipass" \
-e "DB_NAME=wikidb" \
ghcr.io/requarks/wiki:2
```

* Acesse o Wiki.js: `http://localhost:8080/`

## docker-compose

**Atividade 1:**

* Crie uma pasta chamada `dev`: `mkdir dev`

* Acesse a pasta e dentro dela crie o arquivo chamado `docker-compose.yml`:

```
cd dev
nano docker-compose.yml
```
* Cole o seguinte conteúdo:

```
version: "3"
services:
  postgresql:
    container_name: wikijsdb
    hostname: wikijsdb
    restart: always
    volumes:
      - wikijsdb_data:/var/lib/postgresql/data
    networks: 
      - default
    environment:
      POSTGRES_DB: wikidb
      POSTGRES_PASSWORD: wikipass
      POSTGRES_USER: wikiuser
    image: postgres:11-bullseye
  wikijs:
    ports:
        - 8080:3000
    container_name: wikijs
    hostname: wikijs
    restart: always
    volumes:
      - wikijs_config:/wiki/data/content
    networks: 
      - default
    environment:
      DB_TYPE: postgres
      DB_HOST: postgresql
      DB_PORT: 5432
      DB_USER: wikiuser
      DB_PASS: wikipass
      DB_NAME: wikidb
    depends_on:
      - postgresql
    image: requarks/wiki:2

volumes:
  wikijs_config:
  wikijsdb_data:

networks:
 default:
   ```

* Crie o container com volume e rede: `docker-compose up -d`
* Remova o container, volumes e rede: `docker-compose down -v`

**Atividade 2:**

* Crie uma pasta chamada `prod`: `mkdir prod`

* Acesse a pasta e dentro dela crie o arquivo chamado `docker-compose.yml`:

```
cd prod
nano docker-compose.yml
```
* Cole o seguinte conteúdo:

```
version: "3"
services:
  postgres:
    container_name: wikijsdb
    hostname: wikijsdb
    restart: always
    volumes:
      - wikijsdb_data:/var/lib/postgresql/data
    networks: 
      - default
    env_file: .env_postgres
    build: .
    image: wikijsdb:1.0
  wikijs:
    ports:
        - 8080:3000
    container_name: wikijs
    hostname: wikijs
    restart: always
    networks: 
      - default
    env_file: .env_wikijs
    depends_on:
      - postgres
    image: requarks/wiki:2

volumes:
  wikijsdb_data:
    external: true

networks:
 default:
  external:
   name: network-wikijs
   ```  

* Crie os volumes: 

```
docker volume create wikijsdb_data
docker volume create wikijs_config
```

* Crie a rede utilizada:  `docker network create network-wikijs`

* Faça o build da imagem do PostgreSQL com base no Dockerfile: `docker-compose build`

* Crie os container em background: `docker-compose up -d`

* Veja os logs: `docker-compose logs`

# Portainer

**Atividade 1:**

* Crie o volume: `docker volume create portainer_data`
* Crie o container:

```
docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:2.9.3
```

* Liste os containers em execucação: `docker container ls`

* Acesse o Portainer: `https://localhost:9443`
