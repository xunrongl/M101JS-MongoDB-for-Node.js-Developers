# MongoDB Week 1
@(MongoDB)[mongodb]

[TOC]

## Definition
- A document database
- Data Model: JSON
- Easy to distribute and share data across multiple servers --- scaling out, shading feature
- Not support join
- Not support transaction across documents (have to do it manually)

* support common data access patterns
* scalability and performance needs

## Mongo Shell and Node.js High level architecture
## JSON
- array might includes different types of data
### BSON
- Lightweight
- traversable
- efficient (encoding and decoding at driver)
- expand JSON types

## CRUD
### create
db.collectionName.insertOne({object})
### read
- db.collectionName.find({filter}).pretty()
- Note that mongo shell is a javaScript environment, e.g var c =  db.collection.find(); c.hasNext();
### update
### delete

## Node.js Application
``` javascript
var http = require('http');
var server = http.createServer(function (request, response) {
	response.writeHead(200, {"Content-Type": "text/plain"});
	response.end("Hello, World"\n);
});

server.listen(8000);
console.log("Server running at http://localhost:8000")
```

## Node Driver
``` javascript
var MongoClient = require('mongodb').MongoClient,
    assert = require('assert');

var url = 'mongodb://localhost:27017/video';

MongoClient.connect(url, function(err, db) {

    assert.equal(null, err);
    console.log("Successfully connected to server");

    // Find some documents in our collection, return a cursor but here we use native javaScript way to deal with it
    db.collection('movies').find({}).toArray(function(err, docs) {

        // Print the documents returned
        docs.forEach(function(doc) {
            console.log(doc.title);
        });

        // Close the DB
        db.close();
    });

    // Declare success
    console.log("Called find()");
});
```
#### Async
- Instead of waiting any return value, we pass a callback function to handle the result of the operation
- In common callback function, first parameter will be error object and the second one is the resolved object
- the rest of the operation will execute in parallel (in this case, the "Called find()"))

## Express
``` javascript
var express = require('express'),
    app = express();
    //consolidate is a wrapper for templates engines 
    engines = require('consolidate');

//set the html engine
app.engine('html', engines.nunjucks);
//set the view engine
app.set('view engine', 'html');
//__dirname is the path of the current file
app.set('views', __dirname + '/views');

app.get('/', function(req, res){
	//res.send('Hello World');
	res.send('hello', {'name': 'Tom'});
});

app.use(function(req, res){
    res.sendStatus(404); 
});

var server = app.listen(3000, function() {
	//get the port number
    var port = server.address().port;
    console.log('Express server listening on port %s', port);
});
```

### All Together
``` javascript
var express = require('express'),
    app = express(),
    engines = require('consolidate'),
    MongoClient = require('mongodb').MongoClient,
    assert = require('assert');

app.engine('html', engines.nunjucks);
app.set('view engine', 'html');
app.set('views', __dirname + '/views');

MongoClient.connect('mongodb://localhost:27017/video', function(err, db) {

    assert.equal(null, err);
    console.log("Successfully connected to MongoDB.");

    app.get('/', function(req, res){

        db.collection('movies').find({}).toArray(function(err, docs) {
            res.render('movies', { 'movies': docs } );
        });

    });

    app.use(function(req, res){
        res.sendStatus(404);
    });
    
    var server = app.listen(3000, function() {
        var port = server.address().port;
        console.log('Express server listening on port %s.', port);
    });

});
```
### Express handle GET request
``` javascript
app.get('/:name/:test', function(req, res, next) {
    var name = req.params.name;
    var test = req.params.test;
    var getvar1 = req.query.getvar1;
    var getvar2 = req.query.getvar2;
    res.render('hello', { name : name, test : test, getvar1 : getvar1, getvar2 : getvar2 });
});
```
### Express handle POST request
``` javascript
//define this before route
app.use(express.bodyParser());

app.post('/favorite_fruit', function(req, res, next) {
    var favorite = req.body.fruit;
    if (typeof favorite == 'undefined') {
	    //throw error object will find the error handling
        next(Error('Please choose a fruit!'));
    }
    else {
        res.send("Your favorite fruit is " + favorite);
    }
});
```

