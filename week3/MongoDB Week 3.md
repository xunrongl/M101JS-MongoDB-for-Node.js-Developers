#MongoDB Week 3

[TOC]

##node.js driver and CRUD operation
###find
- import data from json file using ```mongoimport```
- assertion
	- ```assert.equal(err, null)```
	- ```assert.notEqual(docs.length, 0)```
- using the cursor, the find method could create a cursor immediately but not making a request to the database until we try to use some of the doc from the database
- cursor.forEach - not the forEach method for array
	- first argu - iterate the doc, streaming the data as we need them (every batch, not the whole data at one time)
	- second argu -  when cursor exhaust and error
	 
###project in node.js driver

```javascript
	var query = {"category_code": "biotech"};
	var projection = {"name": 1, "category_code": 1, "_id": 0};
	var cursor = db.collection('companies').find(query);
	cursor.project(projection);
	cursor.forEach(
	        function(doc) {
	            console.log(doc.name + " is a " + doc.category_code + " company.");
	            console.log(doc);
	        },
	        function(err) {       
	            assert.equal(err, null);
	            return db.close();
	        }
	    );
```
only use the bandwidth and network for the fields of the data that we need

### CLI Argus
```javascript
	var MongoClient = require('mongodb').MongoClient,
	    commandLineArgs = require('command-line-args'), 
	    assert = require('assert');
	
	var options = commandLineOptions();
	
	MongoClient.connect('mongodb://localhost:27017/crunchbase', function(err, db) {
	
	    assert.equal(err, null);
	    console.log("Successfully connected to MongoDB.");
	    
	    var query = queryDocument(options);
	    var projection = {"_id": 1, "name": 1, "founded_year": 1,
	                      "number_of_employees": 1, "crunchbase_url": 1};
	
	    var cursor = db.collection('companies').find(query, projection);
	    var numMatches = 0;
	
	    cursor.forEach(
	        function(doc) {
	            numMatches = numMatches + 1;
	            console.log( doc );
	        },
	        function(err) {
	            assert.equal(err, null);
	            console.log("Our query was:" + JSON.stringify(query));
	            console.log("Matching documents: " + numMatches);
	            return db.close();
	        }
	    );
	
	});
	
	
	function queryDocument(options) {
	
	    console.log(options);
	    
	    var query = {
	        "founded_year": {
	            "$gte": options.firstYear,
	            "$lte": options.lastYear
	        }
	    };
	
	    if ("employees" in options) {
	        query.number_of_employees = { "$gte": options.employees };
	    }
	        
	    return query;
	}
	
	
	function commandLineOptions() {
	
	    var cli = commandLineArgs([
	        { name: "firstYear", alias: "f", type: Number },
	        { name: "lastYear", alias: "l", type: Number },
	        { name: "employees", alias: "e", type: Number }
	    ]);
	    
	    var options = cli.parse()
	    if ( !(("firstYear" in options) && ("lastYear" in options))) {
	        console.log(cli.getUsage({
	            title: "Usage",
	            description: "The first two options below are required. The rest are optional."
	        }));
	        process.exit();
	    }
	
	    return options;   
	}
```
Based on the query above, you can use the application in the following way
``` node app.js -f 2004 -l 2008 -e 1000 ```

### Regular expression
```javascript

	function queryDocument(options) {
	
	    console.log(options);
	    
	    var query = {};
	
	    if ("overview" in options) {
	        query.overview = {"$regex": options.overview, "$options": "i"};
	    }
	
	    if ("milestones" in options) {
	        query["milestones.source_description"] =
	            {"$regex": options.milestones, "$options": "i"};
	    }
	
	    return query;    
	}
	
	function projectionDocument(options) {
	    var projection = {
	        "_id": 0,
	        "name": 1,
	        "founded_year": 1
	    };
	
	    if ("overview" in options) {
	        projection.overview = 1;
	    }
	
	    if ("milestones" in options) {
	        projection["milestones.source_description"] = 1;
	    }
	
	    return projection;
	}
```

### dot notation 
- 3 ways to use javaScript object to define the query
 1. ``` var query = {"founded_year": {}};```
 2. ``` query.number_of_employees = {} ``` use to add field
 3. ``` query["ipo.valuation_amount"] = {} ``` used to add field with dot so that find could use dot notation when query, e.g valuation_amount here is a field of the element of the value of ipo field
 
###Paging method

####skip

```javascript

  //cursor.sort({founded_year: -1});
    cursor.sort([["founded_year", 1], ["number_of_employees", -1]]);

```  
for multiple sort, since there is no guarantee for the fields of the object, so we have to use the array

####limit

```javascript

    cursor.limit(options.limit);
	// in the commanLineOptions function
	var cli = commandLineArgs([
        { name: "firstYear", alias: "f", type: Number },
        { name: "lastYear", alias: "l", type: Number },
        { name: "employees", alias: "e", type: Number },
        { name: "skip", type: Number, defaultValue: 0 },
        { name: "limit", type: Number, defaultValue: 20000 }
    ]);
``` 
 
####sort
```javascript
    cursor.skip(options.skip);
```
MongoDB always sort skip then limit


### Writing data to DB

####insertOne
```javascript
	twitterClient.stream('statuses/filter', {track: "marvel"}, function(stream) {
	        stream.on('data', function(status) {
	            console.log(status.text);
	            db.collection("statuses").insertOne(status, function(err, res) {
	                console.log("Inserted document with _id: " + res.insertedId + "\n");
	            });
	        });
	 
	        stream.on('error', function(error) {
	            throw error;
	        });
	    });
```
###insertMany
```javascript

	 MongoClient.connect('mongodb://localhost:27017/social', function(err, db) {
	
	    assert.equal(null, err);
	    console.log("Successfully connected to MongoDB.");
	
	    var screenNames = ["Marvel", "DCComics", "TheRealStanLee"];
	    var done = 0;
	
	    screenNames.forEach(function(name) {
	
	        var cursor = db.collection("statuses").find({"user.screen_name": name});
	        cursor.sort({ "id": -1 });
	        cursor.limit(1);
	
	        cursor.toArray(function(err, docs) {
	            assert.equal(err, null);
	            
	            var params;
	            if (docs.length == 1) {
	                params = { "screen_name": name, "since_id": docs[0].id, "count": 10 };
	            } else {
	                params = { "screen_name": name, "count": 10 };
	            }
	            
	            client.get('statuses/user_timeline', params, function(err, statuses, response) {
	                
	                assert.equal(err, null);
	                
	                db.collection("statuses").insertMany(statuses, function(err, res) {
	
	                    console.log(res);
	                    
	                    done += 1;
	                    if (done == screenNames.length) {
	                        db.close();
	                    }
	                    
	                });
	            });
	        })
	    });
	});     
```

###delete
####deleteOne

```javascript
MongoClient.connect('mongodb://localhost:27017/crunchbase', function(err, db) {

    assert.equal(err, null);
    console.log("Successfully connected to MongoDB.");
    
    var query = {"permalink": {"$exists": true, "$ne": null}};
    var projection = {"permalink": 1, "updated_at": 1};

    var cursor = db.collection('companies').find(query);
    cursor.project(projection);
    cursor.sort({"permalink": 1})

    var numToRemove = 0;

    var previous = { "permalink": "", "updated_at": "" };
    cursor.forEach(
        function(doc) {

            if ( (doc.permalink == previous.permalink) && (doc.updated_at == previous.updated_at) ) {
                console.log(doc.permalink);
                numToRemove = numToRemove + 1;
                var filter = {"_id": doc._id};

				//delete the first one doc that matches the query or filter
                db.collection('companies').deleteOne(filter, function(err, res) {

                    assert.equal(err, null);
                    console.log(res.result);
                    //{od: 1, n: 1}, ok means delete completed and n means the number of docs that deleted

                });

            }
            
            previous = doc;
            
        },
        function(err) {

            assert.equal(err, null);

        }
    );

});
```
- when you hit the OperationFailed says ```Sort operation used more than the max 335541 byte of == null```, it is because it tries to do sort in memory instead of in the database, so you need to create the index 


####deleteMany
```javascript
MongoClient.connect('mongodb://localhost:27017/crunchbase', function(err, db) {

    assert.equal(err, null);
    console.log("Successfully connected to MongoDB.");
    
    var query = {"permalink": {$exists: true, $ne: null}};
    var projection = {"permalink": 1, "updated_at": 1};

    var cursor = db.collection('companies').find(query);
    cursor.project(projection);
    cursor.sort({"permalink": 1})
	
	//an array to store the docs that you want to remove
    var markedForRemoval = [];

    var previous = { "permalink": "", "updated_at": "" };
    cursor.forEach(
        function(doc) {

            if ( (doc.permalink == previous.permalink) && (doc.updated_at == previous.updated_at) ) {
                markedForRemoval.push(doc._id);
            }

            previous = doc;
        },
        function(err) {

            assert.equal(err, null);

            var filter = {"_id": {"$in": markedForRemoval}};

            db.collection("companies").deleteMany(filter, function(err, res) {

                console.log(res.result);
                console.log(markedForRemoval.length + " documents removed.");

                return db.close();
            });
        }
    );

});
```
- 	use $in operator as the filter
- 	create an array to store the data