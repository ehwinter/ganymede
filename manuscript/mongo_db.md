Mongo DB
===================

(from http://cheat.errtheblog.com/s/mongo)

## CONSOLE COMMANDS
```

  help                show all the console commands (same as this section)
  show dbs            show database names
  show collections    show collections in current database
  show users          show users in current database
  show profile        show most recent system.profile entries with time >= 1ms
  use <db name>       set curent database to <db name>
  db.help()           help on DB methods
  db.foo.help()       help on collection methods
  db.foo.find()       list objects in collection foo
  db.foo.find({a:1})  list objects in foo where a == 1
  it                  result of last line evaluated; use to further iterate
```

## DATABASE METHODS

```
  db.addUser(username, password)
  db.auth(username, password)
  db.cloneDatabase(fromhost)
  db.commandHelp(name) // returns the help for the command
  db.copyDatabase(fromdb, todb, fromhost)
  db.createCollection(name, { size : ..., capped : ..., max : ... } )
  db.currentOp() // displays the current operation in the db
  db.dropDatabase()
  db.eval(func, args) // run code server-side
  db.getCollection(cname) // same as db['cname'] or db.cname
  db.getCollectionNames()
  db.getLastError() // just returns the err msg string
  db.getLastErrorObj() // return full status object
  db.getMongo() // get the server connection object
  db.getMongo().setSlaveOk() // allow this connection to read from the nonmaster member of a replica pair
  db.getName()
  db.getPrevError()
  db.getProfilingLevel()
  db.getReplicationInfo()
  db.getSisterDB(name) // get the db at the same server as this onew
  db.killOp() // kills the current operation in the db
  db.printCollectionStats()
  db.printReplicationInfo()
  db.printSlaveReplicationInfo()
  db.printShardingStatus()
  db.removeUser(username)
  db.repairDatabase()
  db.resetError()
  db.runCommand(cmdObj) // run a database command. if cmdObj is a string, turns it into {cmdObj:1}
  db.setProfilingLevel(level) // 0=off 1=slow 2=all
  db.shutdownServer()
  db.version() // current version of the server
```

## COLLECTION METHODS

```
  db.foo.count()
  db.foo.dataSize()
  db.foo.distinct( key ) // eg. db.foo.distinct( 'x' )
  db.foo.drop() // drop the collection
  db.foo.dropIndex(name)
  db.foo.dropIndexes()
  db.foo.ensureIndex(keypattern,options) // options should be an object with these possible fields: name, unique, dropDups
  db.foo.find( [query] , [fields]) // first parameter is an optional query filter.
                                   // second parameter is optional set of fields to return.
                                   // e.g. db.foo.find( { x : 77 } , { name : 1 , x : 1 } )
  db.foo.find(...).count()
  db.foo.find(...).limit(n)
  db.foo.find(...).skip(n)
  db.foo.find(...).sort(...)
  db.foo.findOne([query])
  db.foo.getDB() // get DB object associated with collection
  db.foo.getIndexes()
  db.foo.group( { key : ..., initial: ..., reduce : ...[, cond: ...] } )
  db.foo.mapReduce( mapFunction , reduceFunction , <optional params> )
  db.foo.remove(query)
  db.foo.renameCollection( newName ) // renames the collection
  db.foo.save(obj)
  db.foo.stats()
  db.foo.storageSize() // includes free space allocated to this collection
  db.foo.totalIndexSize() // size in bytes of all the indexes
  db.foo.totalSize() // storage allocated for all data and indexes
  db.foo.update(query, object[, upsert_bool, multi_record_bool])
  db.foo.validate() // SLOW
  db.foo.getShardVersion() // only for use with sharding
```

## UTILITIES

```
  mongodump -d database_name -o /some/directory
  mongorestore -d database_name /some/directory
```

## EXAMPLES

####  Inserting Data

```
    > j = { name: "mongo"};
    {"name" : "mongo"}
    > t = { x : 3 };
    { "x" : 3  }
    > db.things.save(j);
    > db.things.save(t);
```

####  Accessing Data from a Query

```
    var cursor = db.things.find();
    while (cursor.hasNext()) { print(tojson(cursor.next())); }
```

####  Retrieving Data with Javascript functions

```
    db.things.find().forEach( function(x) { printjson(x);});

  select * from things where name="mongo"
    db.things.find({name:"mongo"}).forEach(printjson);

  select * from things where x=4
    db.things.find({x:4}).forEach(printjson);

  select j from things where x=4
    db.things.find({x:4}, {j:true}).forEach(printjson);
```

####  order by

```
    db.things.find().sort({x:1})  // ascending
    db.things.find().sort({x:-1}) // descending
```

####  limit 1
```

    var mongo = db.things.findOne({name:"mongo"});
    print(tojson(mongo));

  limit(x)
    db.things.find().limit(3);
```

####  regex search
```

    db.things.find({x : /foo.*/i});
```

####  search for type

```
    db.things.find({ x : { $type : 16 }});  // finds integer values in this field

  update users set api_token = 'snafu' where _id = "fubar";
    db.users.update( { "_id" : "fubar" }, { $set : { "api_token" : "snafu" } }, false );

  update or insert users set api_token = 'snafu' where _id = "fubar";
    db.users.update( { "_id" : "fubar" }, { $set : { "api_token" : "snafu" } }, true );

  Rename all fields in a collection
    db.users.update( {}, { $rename : {"old_field_name" : "new_field_name"}}, false, true);
```

## MORE HELP

  In addition to the general "help" command in mongo, you can call help on db
  and db.whatever to see a summary of methods available.

  MongoDB Manual: http://www.mongodb.org/display/DOCS/Manual
