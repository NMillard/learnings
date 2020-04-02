# Mongodb

## Data persistence
At a low level, data is stored as BSON - binary json. Unlike relational databases, mongodb stores data in a schemaless fashion.  
In contrast to table locking as done in RDBMMS, mongodb uses a single document write scope.  

## Writing to database
Documents _must_ have an _id field. If one is not provided, it will get auto-generated.  


## CLI
mongo.exe <- the shell - this is a javascript interpreter  
mongod.exe <- the server  

Mongodb's default data folder is called `mongodbfolder/data/db`.  
When the database folder exists, you can run `mongod` and it will spin up the server.   
Default port is 27017.  

### server configuration
The server can be configured using a configuration file, which is a simple file with keypair values   
```
# mongodb.conf <- file name

dbpath=/some/path/db
logpath/some/logging/path/mongo-server.log
verbose=vvvvv # verbosity goes from 1-5 'v's.
```  

To use the configuration file on server startup, use `mongod -f c:/path/to/file/mongodb.conf`

### install as start-up service
_windows_   
`mongod -f c:/path/to/file/mongodb.conf --install`
`net start mongodb`  


## Simple Commands
`show dbs` shows list of all databases  
`use some_database` switches to the specified database  
`db.products.save({"some": "property"})` save json object in a collection called "products".  

## Replica sets
- Primary database  
    The one and only writeable instance. All clients must be connected the primary if they wish to perform write operations.  
- Secondary  
    Read only instances. There can be many of this type. Data is eventually replicated from the primary database.  
    If the primary database crashes, one of the secondary dbs will become the primary. This allows automatic recovery. 
- Arbiter  
    Used to cast vote on which secondary will become primary in case of failure. The aribter does not hold data.  


## Mongodb in docker

```
# Example docker-commpose.yml file

version: "3"

services:
  mongodb:
    image: mongo:latest
    environment:
      MONGO_INITDB_ROOT_USERNAME: "root"
      MONGO_INITDB_ROOT_PASSWORD: "docker12*!"
    ports:
    - "27017:27017"


# Then just run 'docker-compose up' form where the docker-compose.yml file is located.
```  

You can use the mongo shell by running `docker exec -it nameOfTheContainer mongo`.  
