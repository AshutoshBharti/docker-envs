Based on https://github.com/debezium/debezium-examples/tree/main/tutorial#using-postgres
# Prerequisites
  - Make sure docker has atleast 6G memory and 2 cpu set in docker-desktop resources, else containers will keep dying
# Notes
  - it ll create a docker-vols dir and store volumes data there
  - unless you get a clean start, keep the docker-vols dir clean in your current dir
# Steps
## without schema-registry
  - set debezium version: `export DEBEZIUM_VERSION=1.8`
  - Bring up containers: `docker-compose -f docker-compose.yaml up`
  - open another terminal and from the same dir execute the following
  - Insert some data in postgres: `psql -h localhost -p 15432 -U postgres -d postgres -a -f inventory.sql`
  - start postgres connector: `curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:18083/connectors/ -d @register-postgres.json`
  - Consume messages from a Debezium topic: `docker-compose -f docker-compose-debezium.yaml exec kafka1 /kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092 --from-beginning --property print.key=true  --topic dbserver1.inventory.customers`
  - start another terminal, get inside postgres, insert some new records in inventory schema and see debezium action in above command tail
  - exec into one of the kafka containers and look around:
    - `docker exec -i kafka1 bash`
      - inside container: `./bin/kafka-topics.sh --bootstrap-server kafka1:9092 --list`
  - bring down containers: `docker-compose -f docker-compose.yaml down`


## with schema-registry
  - set debezium version: `export DEBEZIUM_VERSION=1.8`
  - Bring up containers: `docker-compose -f docker-compose-sr.yaml up`
  - open another terminal and from the same dir execute the following
  - Insert some data in postgres: `psql -h localhost -p 15432 -U postgres -d postgres -a -f inventory.sql`
  - start postgres connector: `curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:18083/connectors/ -d @register-postgres-sr.json`
  - Consume messages from a Debezium topic: `docker-compose -f docker-compose-debezium.yaml exec kafka1 /kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092 --from-beginning --property print.key=true  --topic dbserver1.inventory.customers`
  - start another terminal, gt inside postgres, insert some new records in inventory schema and see debezium action in above command tail
  - check schema subjcts; `curl localhost:18081/subjects`
  - exec into one of the kafka containers and look around:
    - `docker exec -i kafka1 bash`
      - inside container: `./bin/kafka-topics.sh --bootstrap-server kafka1:9092 --list`
  - bring down containers: `docker-compose -f docker-compose-sr.yaml down`

