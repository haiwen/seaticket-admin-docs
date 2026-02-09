# Setup SeaTicket Server


## Getting started

The following assumptions and conventions are used in the rest of this document:

- `/opt/seaticket` is the directory for store SeaTicket docker compose files. If you decide to put SeaTicket in a different directory — which you can — adjust all paths accordingly.
- SeaTicket uses some [Docker volumes](https://docs.docker.com/storage/volumes/) for persisting data generated in its database and SeaTicket Docker container. The volumes' [host paths](https://docs.docker.com/compose/compose-file/compose-file-v3/#volumes) like `/opt/caddy-data` and `/opt/seaticket-data`, respectively. It is not recommended to change these paths. If you do, account for it when following these instructions.
- All configuration and log files for SeaTicket and the webserver Nginx are stored in the volume of the SeaTicket container.
- Beside Seadb, SeaTicket server depends on the following third-party services
  - MySQL
  - Redis
  - S3 storage
- SeaTicket contains following components, each packaged as a separate docker image
  - Caddy and seaqa-web
  - seaqa-ai
  - seaqa-events
  - seaqa-indexer


## Deploy SeaTicket

### Deploy Redis

Please refer to [Deploy Redis](https://redis.io/docs/latest/get-started/) for more details.

### Deploy MySQL

Please refer to [Deploy MySQL](https://dev.mysql.com/doc/refman/8.4/en/installing.html) for more details.

Before starting SeaTicket, you need to create MySQL user `seaticket` and create the database `seaticket_db`.

Copy `mysql.sql` file to the host `/tmp` directory:

```bash
docker run --rm -v /tmp:/shared seaticket/seaqa-web:1.0-latest cp /opt/seaticket/seaqa-web/sql/mysql.sql /shared
```

Create the MySQL user `seaticket` and create the database `seaticket_db`:

```bash
mysql -u root -p

# then input the following commands
CREATE DATABASE seaticket_db CHARACTER SET utf8mb4;
CREATE USER seaticket@'%' IDENTIFIED BY <SEAQA_MYSQL_DB_PASSWORD>;
GRANT ALL PRIVILEGES ON seaticket_db.* TO 'seaticket'@'%';
FLUSH PRIVILEGES;

USE seaticket_db;
SOURCE /tmp/mysql.sql;
```

### Download and modify `.env`

To deploy SeaTicket with Docker, you have to `.env`, `caddy.yml`, `seaqa-web.yml`, `seaqa-indexer.yml`, `seaqa-ai.yml`, `seaqa-events.yml`, and `config.yml` in a directory (e.g., `/opt/seaticket`):

```bash
mkdir /opt/seaticket
cd /opt/seaticket

wget -O .env https://manual.seaticket.ai/main/repo/docker/env
wget https://manual.seaticket.ai/main/repo/docker/config.yml
wget https://manual.seaticket.ai/main/repo/docker/caddy.yml
wget https://manual.seaticket.ai/main/repo/docker/seaqa-web.yml
wget https://manual.seaticket.ai/main/repo/docker/seaqa-indexer.yml
wget https://manual.seaticket.ai/main/repo/docker/seaqa-ai.yml
wget https://manual.seaticket.ai/main/repo/docker/seaqa-events.yml

vim .env
```

The following fields merit particular attention:

| Variable                        | Description                                                                                                   | Default Value                   |  
| ------------------------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------- |  
| `SEATICKET_VOLUME`                | The volume directory of SeaTicket data                                                                          | `/opt/seaticket-data`             |  
| `CADDY_VOLUME`          | The volume directory of Caddy data used to store certificates obtained from Let's Encrypt's                    | `/opt/caddy-data`            |  
| `SEATICKET_SERVER_HOSTNAME`       | SeaTicket server hostname or domain                                                                  | (required)  |  
| `SEATICKET_SERVER_PROTOCOL`       | SeaTicket server protocol (http or https)                                                                       | `http` |
| `TIME_ZONE`                     | Time zone                                                                                                     | `UTC`                           |

And modify `config.yml`:

Note: `JWT_PRIVATE_KEY` is the same as the one in SeaDB `.env` file.

The following fields merit particular attention:

| Variable                        | Description                                                                                                   | Default Value                   |  
| ------------------------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------- |  
| `SEAQA_MYSQL_DB_HOST`         | The host of MySQL | `db`  | 
| `SEAQA_MYSQL_DB_PORT`         | The port of MySQL | `3306`  | 
| `SEAQA_MYSQL_DB_USER`         | The user of MySQL |`seaticket`  |  
| `SEAQA_MYSQL_DB_PASSWORD`     | The user `seaticket` password of MySQL                                                                          | (required)  |  
| `SEAQA_MYSQL_SEAQA_DB_NAME`     | The database name of seaticket | `seaticket_db`  |
| `JWT_PRIVATE_KEY`                           | JWT_PRIVATE_KEY, A random string with a length of no less than 32 characters is required for SeaTicket, which can be generated by using `pwgen -s 40 1` | (required) |  
| `REDIS_HOST`       | Redis server host | `redis` |
| `REDIS_PORT`       | Redis server port | `6379` |
| `REDIS_PASSWORD`       | Redis server password | (required) |

### Start SeaTicket server

Modify `COMPOSE_FILE` in `.env` file:

```env
COMPOSE_FILE='caddy.yml,seaqa-web.yml'
```

Start SeaTicket server with the following command

```bash
docker compose up -d
```

!!! warning "ERROR: Named volume "xxx" is used in service "xxx" but no declaration was found in the volumes section"
    You may encounter this problem when your Docker (or docker-compose) version is out of date. You can upgrade or reinstall the Docker service to solve this problem according to the [Docker official documentation](https://docs.docker.com/engine/install/).

!!! note
    You must run the above command in the directory with the `.env`. If `.env` file is elsewhere, please run

    ```sh
    docker compose --env-file /path/to/.env up -d
    ```

### Create an admin

```bash
docker exec -it seaqa-web /scripts/reset-admin.sh
```

Enter the username and password according to the prompts. You now have a new admin account.

Finially, you can go to `http://seaticket.example.com` to use SeaTicket.

### Deploy other components

You can deploy other components of SeaTicket, such as seaqa-ai, seaqa-events, seaqa-indexer.

Copy `/opt/seaticket` to other machines.

Modify `COMPOSE_FILE` in `.env` file:

```env
COMPOSE_FILE='seaqa-ai.yml'
```

Start server with the following command

```bash
docker compose up -d
```

## SeaTicket directory structure

Placeholder spot for shared volumes. You may elect to store certain persistent information outside of a container, in our case we keep various log files and upload directory outside. This allows you to rebuild containers easily without losing important information.

### Path `/opt/seadb-data`

* /opt/seadb-data: This is the directory for seaticket server configuration and data.
    * /opt/seaticket-data/logs: This is the directory that would contain the log files of seaticket server processes. For example, you can find seadb logs in `/opt/seadb/logs/seadb.log`.

### Path `/opt/seaticket-data`

* /opt/seaticket-data: This is the directory for seaticket server configuration and data.
    * /opt/seaticket-data/logs: This is the directory that would contain the log files of seaticket server processes. For example, you can find seaqa-web logs in `/opt/seaticket/logs/seaqa-web.log`.
    * /opt/seaticket-data/seaqa-web-data: This is the directory that would contain the avatar files of seaticket server processes.

### Find logs

To monitor container logs (from outside of the container), please use the following commands:

```bash
# if the `.env` file is in current directory:
docker compose logs --follow
# if the `.env` file is elsewhere:
docker compose --env-file /path/to/.env logs --follow

# you can also specify container name:
docker compose logs seaqa-web --follow
# or, if the `.env` file is elsewhere:
docker compose --env-file /path/to/.env logs seaqa-web --follow
```

The SeaTicket logs are under `/shared/logs` in the docker, or `/opt/seaticket/logs` in the server that run the docker.

To monitor all SeaTicket logs simultaneously (from outside of the container), run

```bash
sudo tail -f $(find /opt/seaticket/logs/ -type f -name *.log 2>/dev/null)
```

## Add a new admin

Ensure the container is running, then enter this command:

```bash
docker exec -it seaqa-web /scripts/reset-admin.sh
```

Enter the username and password according to the prompts. You now have a new admin account.

## Backup and recovery

Follow the instructions in [Backup and restore for SeaTicket Docker](../administration/backup_recovery.md)


## FAQ

### SeaTicket service and container maintenance

Q: If I want enter into the Docker container, which command I can use?

A: You can enter into the docker container using the command:

```bash
docker exec -it seaqa-web /bin/bash
```


Q: I forgot the SeaTicket admin email address/password, how do I create a new admin account?

A: You can create a new admin account by running

```shell
docker exec -it seaqa-web /scripts/reset-admin.sh
```

The SeaTicket service must be up when running the superuser command.


Q: If, for whatever reason, the installation fails, how do I to start from a clean slate again?

A: Remove the directories /opt/seaticket, /opt/seaticket-data and /opt/mysql-data and start again.


Q: Something goes wrong during the start of the containers. How can I find out more?

A: You can view the docker logs using this command: `docker compose logs -f`.
