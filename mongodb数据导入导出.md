Mongodb数据导入导出
===

##使用mongoexport进行数据导出

<pre>
sudo ./mongoexport --help
Export MongoDB data to CSV, TSV or JSON files.

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
  -f [ --fields ] arg                   comma separated list of field names 
                                        e.g. -f name,age
  --fieldFile arg                       file with field names - 1 per line
  -q [ --query ] arg                    query filter, as a JSON string, e.g., 
                                        '{x:{$gt:1}}'
  --csv                                 export to csv instead of json
  -o [ --out ] arg                      output file; if not specified, stdout 
                                        is used
  --jsonArray                           output to a json array rather than one 
                                        object per line
  -k [ --slaveOk ] arg (=1)             use secondaries for export if 
                                        available, default true
  --forceTableScan                      force a table scan (do not use 
                                        $snapshot)
  --skip arg (=0)                       documents to skip, default 0
  --limit arg (=0)                      limit the numbers of documents 
                                        returned, default all
  --sort arg                            sort order, as a JSON string, e.g., 
                                        '{x:1}'

</pre>

- 查看要导出的数据集

<pre>
sudo ./mongo 
MongoDB shell version: 2.6.3
connecting to: test
> show collections
demo
system.indexes
> db.demo.find()
{ "_id" : ObjectId("5407cdb7093df190d3dd6d0f"), "name" : "kobe", "number" : 24, "value" : 87 }
{ "_id" : ObjectId("5407cdc6093df190d3dd6d10"), "name" : "t-mac", "number" : 1, "value" : 65 }
> 
</pre>

- 导出数据到目录文件,json格式

<pre>
sudo ./mongoexport -d test -c demo -o nba.dat

connected to: 127.0.0.1
exported 2 records
</pre>

- 在当前目录查看nba.dat

<pre>
cat nba.dat
{ "_id" : { "$oid" : "5407cdb7093df190d3dd6d0f" }, "name" : "kobe", "number" : 24, "value" : 87 }
{ "_id" : { "$oid" : "5407cdc6093df190d3dd6d10" }, "name" : "t-mac", "number" : 1, "value" : 65 }
</pre>

- 在当前目录查看nba-csv.dat

<pre>
cat nba.dat
{ "_id" : { "$oid" : "5407cdb7093df190d3dd6d0f" }, "name" : "kobe", "number" : 24, "value" : 87 }
{ "_id" : { "$oid" : "5407cdc6093df190d3dd6d10" }, "name" : "t-mac", "number" : 1, "value" : 65 }
</pre>

- 导出数据到目录文件，csv格式

<pre>
cat nba-csv.dat 
name,number,value
"kobe",24.0,87.0
"t-mac",1.0,65.0
</pre>

##使用mongoimport进入数据导入

<pre>
mongoimport --help
Import CSV, TSV or JSON data into MongoDB.

When importing JSON documents, each document must be a separate line of the input file.

Example:
  mongoimport --host myhost --db my_cms --collection docs < mydocfile.json

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
  -f [ --fields ] arg                   comma separated list of field names 
                                        e.g. -f name,age
  --fieldFile arg                       file with field names - 1 per line
  --ignoreBlanks                        if given, empty fields in csv and tsv 
                                        will be ignored
  --type arg                            type of file to import.  default: json 
                                        (json,csv,tsv)
  --file arg                            file to import from; if not specified 
                                        stdin is used
  --drop                                drop collection first 
  --headerline                          first line in input file is a header 
                                        (CSV and TSV only)
  --upsert                              insert or update objects that already 
                                        exist
  --upsertFields arg                    comma-separated fields for the query 
                                        part of the upsert. You should make 
                                        sure this is indexed
  --stopOnError                         stop importing at first error rather 
                                        than continuing
  --jsonArray                           load a json array, not one item per 
                                        line. Currently limited to 16MB.
</pre>

- 利用json格式进行数据导入
<pre>
sudo ./mongoimport -d test -c nba --file nba.dat


sudo ./mongo

MongoDB shell version: 2.6.3
connecting to: test
> db.nba
db.nba
> db.nba.find()
{ "_id" : ObjectId("5407cdb7093df190d3dd6d0f"), "name" : "kobe", "number" : 24, "value" : 87 }
{ "_id" : ObjectId("5407cdc6093df190d3dd6d10"), "name" : "t-mac", "number" : 1, "value" : 65 }
</pre>

- 利用csv格式进行导入 

<pre>
sudo ./mongoimport -d test -c nba-csv --type csv --headerline --file nba-csv.dat
//headerline参数用于跳过开头的一行
</pre>