# Intro
This repository outlines the steps to perform a compaction of the datastore of an AuraDB instance using docker and the neo4j-admin container image

# Prerequisites

Docker installed

Download latest Neo4j Enterprise

`docker pull neo4j:2025.03.0-enterprise`

Download latest Neo4j-Admin

`docker pull neo4j/neo4j-admin`


# Load database from .backup to container

docker run -it --rm \
    --env=NEO4J_ACCEPT_LICENSE_AGREEMENT=yes  \
    -v ${PWD}/data/:/data  \
    -v ${PWD}/backups/my-naura.backup:/backups/my-aura.backup \
    neo4j/neo4j-admin:2025-enterprise \
    neo4j-admin database load --from-path=/backups/ neo4j --overwrite-destination=true --verbose

# Copy and Compact database neo4j

docker run -it --rm \
    --env=NEO4J_ACCEPT_LICENSE_AGREEMENT=yes  \
    -v ${PWD}/data/:/data  \
    neo4j/neo4j-admin:2025-enterprise \
    neo4j-admin database copy neo4j neo4j --compact-node-store=true --verbose 

# Create a new dump

## Start container to validate that the compacted database starts and that the copy operation worked successfully

```
docker run --name=compact-neo4j --detach \
    --publish=7474:7474 --publish=7687:7687 \
    --env NEO4J_ACCEPT_LICENSE_AGREEMENT=yes \
    --env NEO4J_AUTH=neo4j/password \
    -v ${PWD}/data/:/data \
    -v ${PWD}/logs/:/logs \
    neo4j:2025.03.0-enterprise
```
## Stop the container after testsing is completed

`docker stop compact-neo4j`

# Create DUMP

docker run -it --rm \
    --env=NEO4J_ACCEPT_LICENSE_AGREEMENT=yes  \
    -v ${PWD}/data/:/data  \
    neo4j/neo4j-admin:2025-enterprise \
    bin/neo4j-admin database dump neo4j --verbose


# Run neo4j-admin database upload to upload new compacted dump to Neo4j AuraDB instance

docker run -it --rm \
    --env=NEO4J_ACCEPT_LICENSE_AGREEMENT=yes  \
    -v ${PWD}/data/:/data  \
    neo4j/neo4j-admin:2025-enterprise \
    neo4j-admin database upload neo4j --from-path=/data/dumps --to-uri=neo4j+s://48da975f.databases.neo4j.io --overwrite-destination=true --verbose


# Bonus: Automate the process with GitHub Actions

1. Create snapshot

2. download snapshot

3. Run neo4j-admin commands to compact datastore 

3. create .dump file of compacted datastore and export as artifact for following job

4. upload .dump file to AuraDB instance



