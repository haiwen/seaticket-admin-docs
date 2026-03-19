# Rebuild index

## Some commands

### Modify configurations rebuild.sh

```shell

export SEAQA_MYSQL_DB_HOST=xxx
export SEAQA_MYSQL_DB_PORT=3306
export SEAQA_MYSQL_DB_USER=xxx
export SEAQA_MYSQL_DB_PASSWORD=xxx
export SEAQA_MYSQL_DB_NAME=xxx

export SEASEARCH_URL=xxx
export SEASEARCH_TOKEN=xxx
...


```

### Rebuild connection index

```shell
cd /opt/seaqa-indexer/seaqa_indexer/index/script

# rebuild connection normal index
./rebuild_index.sh --connection-id xx --index-type normal clear

./rebuild_index.sh --connection-id xx --index-type normal update

# rebuild connection summary index
./rebuild_index.sh --connection-id xx --index-type summary clear

./rebuild_index.sh --connection-id xx --index-type summary update

# rebuild connection content index
./rebuild_index.sh --connection-id xx --index-type content clear

./rebuild_index.sh --connection-id xx --index-type content update

```

### Rebuild project index

```shell
cd /opt/seaqa-indexer/seaqa_indexer/index/script

# rebuild project normal index
./rebuild_index.sh --project-uuid xxx --index-type normal clear

./rebuild_index.sh --project-uuid xxx --index-type normal update

# rebuild project summary index
./rebuild_index.sh --project-uuid xxx --index-type summary clear

./rebuild_index.sh --project-uuid xxx --index-type summary update

```
