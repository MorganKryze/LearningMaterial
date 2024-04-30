---
title: PostgreSQL with Docker
author: Yann M. Vidamment (MorganKryze)
description: This learning material will guide you through the process of setting up a PostgreSQL database using Docker.
---

# PostgreSQL with Docker

## Introduction

This learning material will guide you through the process of setting up a PostgreSQL database using Docker. Docker is a platform that allows you to develop, ship, and run applications in containers. Containers are lightweight, standalone, and executable packages that include everything needed to run an application, including the code, runtime, system tools, libraries, and settings.

Docker will be particularly convenient compared to a traditional installation of PostgreSQL as it will allow you to run the database in an isolated environment without having to install it on your machine.

## Prerequisites

- A computer.

## Step 1: Install Docker

Docker is available for Windows, macOS, and Linux. You can download the installer from the [official website](https://www.docker.com/products/docker-desktop).

## Step 2: Create a PostgreSQL container

Docker compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services.

Create a new directory named `database` and add a `docker-compose.yml` file with the following content:

```yaml
services:
  db:
    image: postgres:alpine
    restart: always
    hostname: ${DB_HOST}
    env_file:
      - .env
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - ./db:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - ${DB_PORT}:5432

volumes:
  data:
```

Then add a `.env` file in the same directory with the following content:

```env
DB_HOST=localhost
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=mydatabase
DB_PORT=5432
```

Finally add the `init.sql` file in the same directory with the following content:

```sql
CREATE DATABASE mydatabase;
\c mydatabase
CREATE TABLE customer (id SERIAL PRIMARY KEY, name VARCHAR(50));
INSERT INTO customer (name) VALUES ('Alice');
INSERT INTO customer (name) VALUES ('Bob');
```

### Quick explanation

- `image: postgres:alpine`: means that we will run a PostgreSQL container in an `alpine` OS.
- `restart`: if the container stops, will restart automatically.
- `hostname`: the hostname of the container.
- `env_file`: the file to read environment variables from.
- `environment`: the environment variables to set for the postgres.
- `volumes`: `db` will store the data to be persistent and `init.sql` will be executed when the container starts.
- `ports`: the port to expose the PostgreSQL database (here access at <localhost:5432>).

## Step 3: Start the PostgreSQL container

Open a terminal and navigate to the `database` directory. Run the following command to start the PostgreSQL container:

```bash
docker-compose up -d
```

This command will create and start the PostgreSQL container in the background. To verify that the container is running, you can use the following command:

```bash
docker ps
```

You should see a postgres container running.

## Step 4: Access the PostgreSQL database

If you have the tool `psql` installed on your machine, you can connect to the PostgreSQL database using the following command:

```bash
psql -h localhost -U postgres
```

Or instead:

```bash
docker exec -it database-db-1 psql -U postgres
```

## Step 5: Update data manually

You can update the data in the database manually using the `psql` command-line tool. For example, you can insert a new customer into the `customer` table using the following command (after running `docker exec -it database-db-1 psql -U postgres`):

```sql
\c mydatabase
INSERT INTO customer (name) VALUES ('Charlie');
```

## Step 6: Clean up

To stop and remove the PostgreSQL container, you can use the following command:

```bash
docker-compose down
```

However, to erase the persistent data, consider removing the `db` directory.

## To go further

- [PostgreSQL Docker Hub](https://hub.docker.com/_/postgres): official PostgreSQL image on Docker Hub.
- [Docker Compose](https://docs.docker.com/compose/): official documentation for Docker Compose.
