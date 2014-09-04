mongodb数据备份
====

<pre>
mongodump --help
Export MongoDB data to BSON files.

Options:
  --help                                produce help message
  -v [ --verbose ]                      be more verbose (include multiple times
                                        for more verbosity e.g. -vvvvv)
  --quiet                               silence all non error diagnostic 
                                        messages
  --version                             print the program's version and exit
  -h [ --host ] arg                     mongo host to connect to ( <set 
                                        name>/s1,s2 for sets)
  --port arg                            server port. Can also use --host 
                                        hostname:port
  --ipv6                                enable IPv6 support (disabled by 
                                        default)
  -u [ --username ] arg                 username
  -p [ --password ] arg                 password
  --authenticationDatabase arg          user source (defaults to dbname)
  --authenticationMechanism arg (=MONGODB-CR)
                                        authentication mechanism
  --gssapiServiceName arg (=mongodb)    Service name to use when authenticating
                                        using GSSAPI/Kerberos
  --gssapiHostName arg                  Remote host name to use for purpose of 
                                        GSSAPI/Kerberos authentication
  --dbpath arg                          directly access mongod database files 
                                        in the given path, instead of 
                                        connecting to a mongod  server - needs 
                                        to lock the data directory, so cannot 
                                        be used if a mongod is currently 
                                        accessing the same path
  --directoryperdb                      each db is in a separate directory 
                                        (relevant only if dbpath specified)
  --journal                             enable journaling (relevant only if 
                                        dbpath specified)
  -d [ --db ] arg                       database to use
  -c [ --collection ] arg               collection to use (some commands)
  -o [ --out ] arg (=dump)              output directory or "-" for stdout
  -q [ --query ] arg                    json query
  --oplog                               Use oplog for point-in-time 
                                        snapshotting
  --repair                              try to recover a crashed database
  --forceTableScan                      force a table scan (do not use 
                                        $snapshot)
  --dumpDbUsersAndRoles                 Dump user and role definitions for the 
                                        given database
</pre>

##使用mongodump备份数据

<pre>
//将test数据库存放在当前目录的test文件中
sudo ./mongodump -d test -o test

得到如下数据文件,其中bson为数据文件，json为元数据文件
-rw-r--r-- 1 root root  137 Sep  4 10:54 demo.bson
-rw-r--r-- 1 root root   91 Sep  4 10:54 demo.metadata.json
-rw-r--r-- 1 root root  121 Sep  4 10:54 nba.bson
-rw-r--r-- 1 root root  137 Sep  4 10:54 nba-csv.bson
-rw-r--r-- 1 root root   94 Sep  4 10:54 nba-csv.metadata.json
-rw-r--r-- 1 root root   90 Sep  4 10:54 nba.metadata.json
-rw-r--r-- 1 root root  194 Sep  4 10:54 system.indexes.bson
</pre>

##数据恢复mongorestore

<pre>
 mongorestore --help
Import BSON files into MongoDB.

usage: mongorestore [options] [directory or filename to restore from]
Options:
  --help                                produce help message
  -v [ --verbose ]                      be more verbose (include multiple times
                                        for more verbosity e.g. -vvvvv)
  --quiet                               silence all non error diagnostic 
                                        messages
  --version                             print the program's version and exit
  -h [ --host ] arg                     mongo host to connect to ( <set 
                                        name>/s1,s2 for sets)
  --port arg                            server port. Can also use --host 
                                        hostname:port
  --ipv6                                enable IPv6 support (disabled by 
                                        default)
  -u [ --username ] arg                 username
  -p [ --password ] arg                 password
  --authenticationDatabase arg          user source (defaults to dbname)
  --authenticationMechanism arg (=MONGODB-CR)
                                        authentication mechanism
  --gssapiServiceName arg (=mongodb)    Service name to use when authenticating
                                        using GSSAPI/Kerberos
  --gssapiHostName arg                  Remote host name to use for purpose of 
                                        GSSAPI/Kerberos authentication
  --dbpath arg                          directly access mongod database files 
                                        in the given path, instead of 
                                        connecting to a mongod  server - needs 
                                        to lock the data directory, so cannot 
                                        be used if a mongod is currently 
                                        accessing the same path
  --directoryperdb                      each db is in a separate directory 
                                        (relevant only if dbpath specified)
  --journal                             enable journaling (relevant only if 
                                        dbpath specified)
  -d [ --db ] arg                       database to use
  -c [ --collection ] arg               collection to use (some commands)
  --objcheck                            validate object before inserting 
                                        (default)
  --noobjcheck                          don't validate object before inserting
  --filter arg                          filter to apply before inserting
  --drop                                drop each collection before import
  --oplogReplay                         replay oplog for point-in-time restore
  --oplogLimit arg                      include oplog entries before the 
                                        provided Timestamp (seconds[:ordinal]) 
                                        during the oplog replay; the ordinal 
                                        value is optional
  --keepIndexVersion                    don't upgrade indexes to newest version
  --noOptionsRestore                    don't restore collection options
  --noIndexRestore                      don't restore indexes
  --restoreDbUsersAndRoles              Restore user and role definitions for 
                                        the given database
  --w arg (=0)                          minimum number of replicas per write
</pre>

- 将备份过的数据，导入到新的数据库 teststore

<pre>
sudo ./mongorestore -d teststore test/test

sudo ./mongo
MongoDB shell version: 2.6.3
connecting to: test
> show dbs
admin      (empty)
local      0.078GB
test       0.078GB
teststore  0.078GB
> 
</pre>