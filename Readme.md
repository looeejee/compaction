# Intro
This repository outlines the steps to perform a compaction of the datastore of an AuraDB instance using docker and the neo4j-admin container image

For this test we are using the Movie Recommendation template

![Movie Reccomendation Database](/img/movie_database.png)

# Prerequisites

Docker installed

Download latest Neo4j Enterprise

`docker pull neo4j:2025.03.0-enterprise`

Download latest Neo4j-Admin

`docker pull neo4j/neo4j-admin`

# Initial Status

First we will connnect to the AuraDB isntance and run the command `:sysinfo` to gather information on the current Datastore Size

![:sysinfo output initial](/img/sysinfo_ouput__initial_aura.png)

Then we will delete some relationships in our Graph

![deleting relationshios](/img/delete_relationships.png)

Then we can run `:sysinfo` again and notice how, even if number of relationships has decreased, the total Datastore Size remain unchanged

![:sysinfo after deleting relationships](/img/sysinfo_ouput_pre_compaction_aura_2.png)


# Create backup of the AuraDB instance by creating an on demand snapshot (From Aura Console)

![Create Snapshot](/img/create_snapshot.png)

Save the Snapshot in the `backups` folder

# Load database from .backup to container

Execute the following command to load the .backup to the container

```
docker run -it --rm \
    --env=NEO4J_ACCEPT_LICENSE_AGREEMENT=yes  \
    -v ${PWD}/data/:/data  \
    -v ${PWD}/backups/my-naura.backup:/backups/my-aura.backup \
    neo4j/neo4j-admin:2025-enterprise \
    neo4j-admin database load --from-path=/backups/ neo4j --overwrite-destination=true --verbose
```

# Copy and Compact database neo4j

```
docker run -it --rm \
    --env=NEO4J_ACCEPT_LICENSE_AGREEMENT=yes  \
    -v ${PWD}/data/:/data  \
    neo4j/neo4j-admin:2025-enterprise \
    neo4j-admin database copy neo4j neo4j --compact-node-store=true --verbose 
```
# Create a new dump

### Optional: Start container to validate that the compacted database starts and that the copy operation worked successfully

```
docker run --name=compact-neo4j --detach \
    --publish=7474:7474 --publish=7687:7687 \
    --env NEO4J_ACCEPT_LICENSE_AGREEMENT=yes \
    --env NEO4J_AUTH=neo4j/password \
    -v ${PWD}/data/:/data \
    -v ${PWD}/logs/:/logs \
    neo4j:2025.03.0-enterprise
```

![Review Datastore Size after comaction (locally)](/img/sysinfo_ouput_post_compaction_local.png)


### Stop the container after testsing is completed

`docker stop compact-neo4j`



# Create DUMP

Use the following command to create the new .dump file

```
docker run -it --rm \
    --env=NEO4J_ACCEPT_LICENSE_AGREEMENT=yes  \
    -v ${PWD}/data/:/data  \
    neo4j/neo4j-admin:2025-enterprise \
    bin/neo4j-admin database dump neo4j --verbose
```

# Run neo4j-admin database upload to upload new compacted dump to Neo4j AuraDB instance
```
docker run -it --rm \
    --env=NEO4J_ACCEPT_LICENSE_AGREEMENT=yes  \
    -v ${PWD}/data/:/data  \
    neo4j/neo4j-admin:2025-enterprise \
    neo4j-admin database upload neo4j --from-path=/data/dumps --to-uri=neo4j+s://48da975f.databases.neo4j.io --overwrite-destination=true --verbose
```

![Verify Compacted Datastore Size in Aura](/img/sysinfo_output_post_compaction_aura.png)

# Bonus: Automate the process with GitHub Actions

1. Create snapshot

2. download snapshot

3. Run neo4j-admin commands to compact datastore 

3. create .dump file of compacted datastore and export as artifact for following job

4. upload .dump file to AuraDB instance



