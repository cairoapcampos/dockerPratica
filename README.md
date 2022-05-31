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
requarks/wiki:2
```

* Inspecione o container wikijsdb, o seu volume network-wikijs e a rede network-wikijs:

```
docker inspect wikijs
docker inspect wikijsdb_data
docker inspect network-wikijs
```

* Liste os containers criados: `docker ps` ou `docker container ls`

* Liste os volumes criados: `docker volume ls`

* Liste a rede criada: `docker network ls`

* Liste as imagens baixadas: `docker image ls`

* Crie e copie um arquivo para o `wikijsdb`: 

```
touch teste.txt
docker cp teste.txt wikijsdb:/
```

* Acesse o container `wikijsdb`: `docker exec -it wikijsdb /bin/bash`

* Veja o log do container `wikijs`: `docker container logs wikijs`

* Acesse o Wiki.js: `http://localhost:8080/`

* Remova os conatainers:

```
docker container stop wikijs wikijsdb
docker container ls -a
docker container stop wikijs wikijsdb
```
**Obs**: `docker container ls -a` e `docker ps -a` são equivalentes.

* Remova os volumes: `docker volume rm wikijs_config wikijsdb_data`

* Remova a rede criada: `docker network rm network-wikijs`

* Remova as imagens: `docker image rm postgres:11-bullseye requarks/wiki:2`


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
  postgres:
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
      DB_HOST: postgres
      DB_PORT: 5432
      DB_USER: wikiuser
      DB_PASS: wikipass
      DB_NAME: wikidb
    depends_on:
      - postgres
    image: requarks/wiki:2

volumes:
  wikijs_config:
  wikijsdb_data:

networks:
 default:
   ```

* Crie o container com volume e rede: `docker-compose up -d`

* Liste os containers: docker-compose ps

* Visualize os logs dos dois containers:

```
docker-compose logs postgres
docker-compose logs wikijs
```
**Obs:** `docker-compose logs` possibilita visualizar logs de todos os containers criados.

* Remova os containers, volumes e rede: `docker-compose down -v`

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
    volumes:
      - wikijs_config:/wiki/data/content
    networks: 
      - default
    env_file: .env_wikijs
    depends_on:
      - postgres
    image: requarks/wiki:2

volumes:
  wikijsdb_data:
    external: true
  wikijs_config:
    external: true

networks:
 default:
  external:
   name: network-wikijs
   ```  
 
* Dentro da mesma pasta crie o arquivo chamado `Dockerfile`: `nano Dockerfile`

* Cole o seguinte conteúdo:

```
FROM postgres:11-bullseye
LABEL maintainer="Cairo Aparecido Campos <cairoapcampos@gmail.com>"

# Atualiza a lista de repositórios
RUN apt-get update

# Insta os pacotes necessários e remove pacotes desnecessários
RUN apt-get install -y iproute2 iputils-ping \
    && apt-get clean \
    && apt-get autoremove \
    && rm -rf /var/lib/apt/lists/*
```

* Dentro da mesma pasta crie o arquivo chamado `.env_postgres`: `nano .env_postgres`

* Cole o seguinte conteúdo:

```
POSTGRES_DB=wikidb
POSTGRES_PASSWORD=wikipass
POSTGRES_USER=wikiuser
```
* Dentro da mesma pasta crie o arquivo chamado `.env_wikijs`: `nano .env_wikijs`

* Cole o seguinte conteúdo:

```
DB_TYPE=postgres
DB_HOST=wikijsdb
DB_PORT=5432
DB_USER=wikiuser
DB_PASS=wikipass
DB_NAME=wikidb
```

* Crie os volumes: 

```
docker volume create wikijsdb_data
docker volume create wikijs_config
```

* Crie a rede utilizada:  `docker network create network-wikijs`

* Faça o build da imagem do PostgreSQL com base no Dockerfile: `docker-compose build`

* Visualize a imagem criada: `docker image ls`

* Crie os container em background: `docker-compose up -d`

* Veja os logs: `docker-compose logs`

* Acesse o Wiki.js (http://localhost:8080/)  e faça a configuração inicial. Depois criei uma página inicial em Markdown com o código abaixo e salve:

```
# Teste
Hello, world! :beers:
```
Como irá ficar:

![alt text](https://github.com/cairoapcampos/dockerPratica/blob/main/img.png)

* Remova os containers: `docker-compose down -v`

* Crie os container novamente em background: `docker-compose up -d`

Será que a mensagem criada ainda está lá ? 

# Portainer

**Atividade 1:**

* Crie o volume: `docker volume create portainer_data`
* Crie o container:

```
docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:2.13.1
```

* Liste os containers em execucação: `docker container ls`

* Acesse o Portainer: `https://localhost:9443`
