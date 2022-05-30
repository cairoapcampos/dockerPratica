# dockerPratica
Repositório com exemplos de docker e docker-compose

1. Criar um arquivo chamado `exer1.yml` com o seguinte conteúdo:

```
version: "3"
services:
  postgres:
    restart: always
    volumes:
      - /data/wikijsdb_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: wikidb
      POSTGRES_PASSWORD: wikipass
      POSTGRES_USER: wikiuser
    image: postgres:11-bullseye
  wikijs:
    ports:
        - 8080:3000
    restart: always
    volumes:
      - /data/wikijs_config:/wiki/data/content
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
   ```
