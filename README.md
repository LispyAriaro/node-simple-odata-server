# Node simple OData server, with Power Bi support

**Super simple implementation of OData server running on Node.js with easy adapters for mongodb and nedb. It will work with Power Bi** Just define an OData model, provide a mongo or nedb database, hook into node.js http server and run. It

It supports basic operations you would expect like providing $metadata, filtering and also operations for insert, update and delete. On the other hand it suppose to be really simple so you don't get support for entity links, batch operations and many others.

The implementation is tested with [.net OData client](https://github.com/object/Simple.OData.Client) and it should fulfill basic protocol requirements.

## Get started

This is how you can create an OData server with node.js http module and nedb.
```js
var http = require('http');
var Datastore = require('nedb');
var db = new Datastore( { inMemoryOnly: true });
var ODataServer = require("simple-odata-server");

var model = {
    namespace: "jsreport",
    entityTypes: {
        "UserType": {
            "_id": {"type": "Edm.String", key: true},
            "test": {"type": "Edm.String"},            
        }
    },   
    entitySets: {
        "users": {
            entityType: "jsreport.UserType"
        }
    }
};

var odataServer = ODataServer("http://localhost:1337")
    .model(model)
    .onNeDB(function(es, cb) { cb(null, db)});


http.createServer(odataServer.handle.bind(odataServer)).listen(1337);
```

Now you can try requests like:<br/>
GET [http://localhost:1337/$metadata]()<br/>
GET [http://localhost:1337/users?$filter=test eq 'a' or test eq 'b'&$skip=1&$take=5]()<br/>
GET [http://localhost:1337/users('aaaa')]()<br/>
GET [http://localhost:1337/users?$orderby=test desc]()<br/>
GET [http://localhost:1337/users/$count]()<br/>
POST, PATCH, DELETE

## mongodb
It works the same way with nedb and mongo. You just need to provide callback for mongo database instance.
```js
MongoClient.connect(url, function(err, db) {
	odataServer.onMongo(function(cb) { cb(err, db); }); 
});
```

## express.js
It works well also with the express.js. You even don't need to provide service uri in the `ODataServer` constructor because it is taken from the express.js request.

```js
app.use("/odata", function (req, res) {
        odataServer.handle(req, res);
    });
```

## cors
You can quickly set up cors without using express and middlewares using this call

```js
odataServer.cors('*')
```

## Configurations
Using `onNeDB` and `onMongo` is just a simple way for initializing `ODataServer`. You can implement your own data layer or override default behavior using following methods: 

```js
odataServer
	.query(fn(setName, query, cb))
	.update(fn(setName, query, update, cb))
	.insert(fn(setName, doc, cb))
	.remove(fn(setName, query, cb))
	.beforeQuery(fn(setName, query, req, cb))
	.beforeUpdate(fn(setName, query, req, update))
	.beforeInsert(fn(setName, doc, req, cb))
	.beforeRemove(fn(setName, query, req, cb))
	.afterRead(fn(setName, result));
	//add hook to error which you can handle or pass to default
	.error(fn(req, res, error, default))
```



## Contributions
I will maintain this repository for a while because I use it in [jsreport](https://github.com/jsreport/jsreport). You are more than welcome to contribute with pull requests and add other basic operations you require. 

## Limitations
- document ids must have name **_id**
- no entity links
- no batch operations
- no validations
- ... this would be a very long list, so rather check yourself

## License
See [license](https://github.com/pofider/node-simple-odata-server/blob/master/LICENSE)

