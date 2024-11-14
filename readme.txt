#verfiy postgres wal level is set to logical
docker exec debezium-postgres psql --username=postgres --dbname=debezium-db --command='SHOW wal_level'

#verify postgres shared_preload_libraries is set to decoderbufs
docker exec debezium-postgres psql --username=postgres --dbname=debezium-db --command='SHOW shared_preload_libraries'

#configure debezium has 2 options :
# OPTION 1 debezium server
# standalone debezium with more powerful features ( can push changes to webhook,...)
# reference : https://debezium.io/documentation/reference/stable/operations/debezium-server.html
update the config files inside ./debezium/conf/application.properties as needed

# OPTION 2 debezium postgres connector
# all changes in source will create new payload in kafka with topics value of topic.prefix+table_name
# to set the connector configuration send http payload into the connector service. sample below :
# add postgres connector
curl --location 'http://localhost:8083/connectors' \
   --header 'Accept: application/json' \
   --header 'Content-Type: application/json' \
   --data '{
    "name": "inventory-connector",
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "database.server.name":"DEBEZIUM POSTGRES",
        "database.hostname": "debezium-postgres",
        "database.port": "5432",
        "database.user": "postgres",
        "database.password": "123",
        "database.dbname": "debezium-db",
        "topic.prefix": "pg-mac",
        "schema.include.list": "public",
        "plugin.name": "pgoutput",
        "table.include.list": "public.user"
    }
}'
# to update connector
curl --location --request PUT 'localhost:8083/connectors/inventory-connector/config' \
--header 'Content-Type: application/json' \
--data '{
  "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
  "database.user": "postgres",
  "database.dbname": "debezium-db",
  "tasks.max": "1",
  "database.server.name": "DEBEZIUM POSTGRES",
  "schema.include.list": "public",
  "database.port": "5432",
  "plugin.name": "pgoutput",
  "key.converter.schemas.enable": "false",
  "topic.prefix": "pg-mac",
  "database.hostname": "debezium-postgres",
  "database.password": "123",
  "value.converter.schemas.enable": "false",
  "transforms.changes.type":"io.debezium.transforms.ExtractChangedRecordState"
  debezium.sink.type=http

#notes if need to capture data before changes alter the table add replica identity to full :
ex: ALTER TABLE your_table_name REPLICA IDENTITY FULL;