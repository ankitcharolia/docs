---
layout: default
title: CouchDB Operations
parent: Database
nav_order: 3
---

### List all Databases
```shell
curl -X GET http://admin:oasjinjksf@127.0.0.1:5984/_all_dbs
```

### Create a Database
```shell
curl -X PUT http://admin:oasjinjksf@127.0.0.1:5984/<DB_NAME>
```

### Export a Database
```shell
curl -X GET http://admin:oasjinjksf@127.0.0.1:5984/<DB_NAME>/_all_docs?include_docs=true > app-user.json
```

### Transform the exported json file (The following format is needed for importing the database)
#### modify the exported json file to look like something like the json below (note the _id):
```shell
{
"docs": [
{"_id": "0", "integer": 0, "string": "0"},
{"_id": "1", "integer": 1, "string": "1"},
{"_id": "2", "integer": 2, "string": "2"}
]
}
```

```shell
# create a file called transformdocs.js
var fs = require("fs");
console.log("\n *START transformation* \n");

var content = fs.readFileSync("app-user.json");
var json = JSON.parse(content);
var docs = json.rows;
var newDocs = new Array();

docs.forEach(function(doc) {
var innerdoc = doc.doc;
delete innerdoc._rev;
newDocs.push(innerdoc);
});

var newJson = new Object();
newJson.docs = newDocs;
var newContent = JSON.stringify(newJson)
fs.writeFile('app-user-modified.json', newContent, 'utf8', function(err) {
if (err) throw err;
console.log('complete');
});
console.log("\n *DONE transformation!* \n");
```

```shell
# execute the file to transform the json
node transformdocs.js
```

### Import a Database
```shell
curl -d @app-user-modified.json -H "Content-type: application/json" -X POST http://admin:oasjinjksf@127.0.0.1:5984/<DB_NAME>/_bulk_docs
```

### Create a User (user: jan)
```shell
curl -X PUT http://admin:oasjinjksf@127.0.0.1:5984/_users/org.couchdb.user:jan -H "Accept: application/json" -H "Content-Type: application/json" -d '{"name": "jan", "password": "apple", "roles": [], "type": "user"}'
Output: {"ok":true,"id":"org.couchdb.user:jan","rev":"1-9be9fd6b312ed8ebc31ea940c0b21c80"}
```

### List all users
```shell
curl http://admin:oasjinjksf@127.0.0.1:5984/_users/_all_docs
{"total_rows":2,"offset":0,"rows":[
{"id":"_design/_auth","key":"_design/_auth","value":{"rev":"1-753ae0157a8b1a22339f3c0ef4f1bf19"}},
{"id":"org.couchdb.user:jan","key":"org.couchdb.user:jan","value":{"rev":"1-9be9fd6b312ed8ebc31ea940c0b21c80"}}
]}
```

### Check if users exists
```shell
curl -X POST http://admin:oasjinjksf@127.0.0.1:5984/_session -d 'name=jan&password=apple'
Output: {"ok":true,"name":"jan","roles":[]}
```

## Grant Authorization
### Declare user: jan as a member of a database
```shell
curl -X PUT http://admin:oasjinjksf@127.0.0.1:5984/<DB_NAME>/_security -H "Content-Type: application/json" -d '{"admins": { "names": [], "roles": [] }, "members": { "names": ["jan"], "roles": [] } }'
{"ok":true}
```

### check if user can access the database now
```shell
curl -X GET http://<DB_USER>:<DB_PASSWORD>@127.0.0.1:5984/<DB_NAME>
```
