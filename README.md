# Docker Compose files

## Created by:

- Ariel Duarte (c)2022

### Run docker docker compose file

```bash
$ docker-compose up -d
```

### Preparar imagen de Docker - Node App 

## Build
docker-compose -f docker-compose.prod.yaml --env-file .env.prod up --build

## Run
docker-compose -f docker-compose.prod.yaml --env-file .env.prod up

## Nota
Por defecto, __docker-compose__ usa el archivo ```.env```, por lo que si tienen el archivo .env y lo configuran con sus variables de entorno de producción, bastaría con
```bash
docker-compose -f docker-compose.prod.yaml up --build
```

### docker-compose.prod.yaml 

```js
version: '3'

services:
  [app_name]:
    depends_on:
      - db
    build: 
      context: .
      dockerfile: Dockerfile
    image: [project_name]-docker
    container_name: [app_name]
    restart: always # reiniciar el contenedor si se detiene
    ports:
      - "${PORT}:${PORT}"
    # working_dir: /var/www/[project_name]
    environment:
      MONGODB: ${MONGODB}
      PORT: ${PORT}
      DEFAULT_LIMIT: ${DEFAULT_LIMIT}
    # volumes:
    #   - ./:/var/www/[project_name]

  db:
    image: mongo:5
    container_name: [container-name]
    restart: always
    ports:
      - 27017:27017
    environment:
      MONGODB_DATABASE: [mongo-database-name]
    # volumes:
    #   - ./mongo:/data/db
```

### Dockerfile 

```
# Install dependencies only when needed
FROM node:18-alpine3.15 AS deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn install --frozen-lockfile

# Build the app with cache dependencies
FROM node:18-alpine3.15 AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN yarn build


# Production image, copy all the files and run next
FROM node:18-alpine3.15 AS runner

# Set working directory
WORKDIR /usr/src/app

COPY package.json yarn.lock ./

RUN yarn install --prod

COPY --from=builder /app/dist ./dist

# # Copiar el directorio y su contenido
# RUN mkdir -p ./[project_name]

# COPY --from=builder ./app/dist/ ./app
# COPY ./.env ./app/.env

# # Dar permiso para ejecutar la applicación
# RUN adduser --disabled-password [user_name]
# RUN chown -R [user_name]:[user_name] ./pokedex
# USER [user_name]

# EXPOSE 3000

CMD [ "node","dist/main" ]
```

### Docker simple file

```
FROM node:18-alpine3.15

# Set working directory
RUN mkdir -p /var/www/[project_name]
WORKDIR /var/www/[project_name]

# Copiar el directorio y su contenido
COPY . ./var/www/[project_name]
COPY package.json tsconfig.json tsconfig.build.json /var/www/[project_name]/
RUN yarn install --prod
RUN yarn build


# Dar permiso para ejecutar la applicación
RUN adduser --disabled-password pokeuser
RUN chown -R [user_name]:[user_name] /var/www/[project_name]
USER [user_name]

# Limpiar el caché
RUN yarn cache clean --force

EXPOSE 3000

CMD [ "yarn","start" ]
```
