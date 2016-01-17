#MongoDB Week 2

[TOC]

##CRUD
### Creating Document
- insert()
 - you can specify _id as well
- insertMany()
 - passing an array to list as many element as you like
 - as soon as it encounters an error, it will stop
 - if you want it to still insert, just specify ``` {"ordered": false}``` 
- Update Commands ("upserts"), if upserts does not mach any records

### _id
ObjectId (12-byte HEX String): Date (4 bytes) + MAC address(3 bytes) + PID (2 bytes) + Counter (3 bytes)

### Reading Document
- find()
 - use dot notation ``` find({"tomato.meter": 100})```
#### equality matches on Arrays 
- on the entire array
 - the order of the array matters ``` {"writes": ["Ethan", "Joel"]}```
- based on any element  ``` {"actors": "Mark"}```
- based on a special element ``` {"actors.0": "Jeff"}```
- more complex matches using operators

#### cursors
``` javascript
 var c = db.movieDetails.find();
 var doc = function() { return c.hasNext() ? c.next() : null; }
```
 - ```c.objsLeftInBatch()```

#### projection
second argument of the find command to specify the fields that you want from the documents

```javascript
find({},{title:1, _id: 0})
find({},{writes:0, _id: 0})
```

### Query Selectors
- \$eq, \$ne
- \$gt, \$gte, \$lt, \$lte
- \$in ``` {"rated": {$in: ["G", "PG-13"]}}```, \$nin
- \$exists ``` {"tomato.meter": {$exists: false}}```
- \$type ``` {"_id": {$type: "string"}}```
- \$or, \$and, \$not, \$nor ```{$or: [{"tomato.meter": {$gt: 95}}, {"metacritic": {$gt: 88}}]}```
 - for $and, it's used for specify same column more than once ```{$and:[{"metacritic": {$ne: null}}, {"metacritic": {$exists: true}}]}``` 
- $regex ```{"awards.text: {$regex: /^Won\s.*/ }"}```
#### Array query selctors
- $all ``` {genres: {$all: ["Comedy", "Crime"]}}```
- $size ```{contries:{$size: 1}}```
- $elemMatch 
``` javascript
//matches the boxOffice array in which element has a "UK" 
//and element(could be another element) has a revenue > 15
{boxOffice: {country: "UK", revenue: { $gt: 15}}} 
//matches the boxOffice array in which an only single element 
//has both a "UK" and revenue > 15
{boxOffice: {$elemMatch: {"country": "USA", revenue: {$gt: 15}}} }
```

### Update documents
#### updateOne
- $set 

```javascript
updateOne({title: "The Martian"}, {$set: {poster: "http://..."}})
```
 - $inc, increment the value by
 
 ``` javascript
updateOne({title: "The Martian"}, {$inc: {""reviews": 3}) 
 ```
 
Array update operators
 - $push
  - $each: push to the "review"array instead of create a push a new "review" array with one single element
  - $slice: keep just the number (first/last) element
  - $position: 0 make sure push the first position
```javascript
updateOne({title: "The Martian"},
		{$push: {reviews:
				{$each: [
					{rating: 0.5,
					date: ISODate(""),
					reviewer: "XXX",
					text: "XXX"}],
				$position: 0,
				$slice: 5 }}})
```
#### updateMany
``` javascript
//make sure only the document with rate (e.g PG, PG-13, UNRATED) would have rate field
updateMany({rated: null}, {$unset: {rated: ""}})
```

#### Upserts
```javascript
updateOne(
{"imdb.id": detail.imdb.id},
{$set: detail}, //guarantees that I won't create a second doc
{upsert: true}); //if the filter doesnt match any doc in the collections, I want to go ahead and insert it
```
Note:
>upsert是一个选项，它是update的第三个参数，并不是一个方法。它是一种特殊的更新，要是没有文档符合匹配，那么它就会根据条件和更新文档为基础，创建新的文档，如有匹配，则正常更新。咱们之前见到的所有update操作，都是建立在有文档的基础之上的。upsert非常方便，不必预制集合，同一套代码既可以创建又可以更新。

#### ReplaceOne
``` javascript
replaceOne(
	{"imdb.id": detail.imdb.id},
	detail);
```