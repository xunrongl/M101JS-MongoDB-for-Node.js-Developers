MongoDB Week 4

[TOC]

#MongoDB Schema Design

## MongoDB features
- Rich Documents 
- Pre Join/Embed Data 
 - the data that you use together you might embed one to the other
- Not support join
- No constraints (not foreign key)
- Support Atomic operations but not transaction 
- No Declared Schema
In summary, matching the pattern that you access the data 

## Relational 3rd normal form
an example that violate the form:
| ID | Author | email  |
| - |-| -----:|
| 1 | Andrew | andrew@gmail.com |
| 2 | Richard | richard@gmail.com |
| 3 | Andrew | andrew@gmail.com |
if I modify the email of first item, the last one may be inconsistent, which violate normalization

###Goals of Normalization
- free the database of mdification anomalies
- Minimize redesign when extending
- Avoid bias toward any particular access pattern

###Model a blog in MongoDB
```javascript
Post
{
	_id:"",
	title:"",
	content:"",
	author:"",
	comments:[
		{
			author:"",
			comment:"",
			date:Date()
		}
	],
	tags:["",""],
	date:Date()
}
Author
{
	_id:"",
	name:"",
	email:"",
	password:""
}
```
###Living without constraints
Relational database maintain the data consistency via foreign key
MongoDB use **Embed**


###Living without transaction
MongoDB supports Atomic operations, which means
- when you work on a single document, that work will be completed before anyone else can see the document
- using atomic operations, you can accomplish same thing that you use transaction in the relational database
	- in relational db, when you do operations across the table, you have to use joins
	- with MongoDB, because of the prejoin/embed
 	 1. restructure so that you work within a single document
 	 2. implement Locking in Software (SLO)
 	 3. tolerate the inconsistency 


## One to One Relations
Employee: Resume
for MongoDB, you can have the link with key or embed, depends on:
- Frequency of access
 - if you read employee very often but resume very seldom, you might keep them as two separate collections, because you don't want to pull the resume into memory every single time that you pull the employee
- Size of items
 - if you never update the Employee but you might update Resume, you might decide to keep them separately (so that you don't have to update twice)
 - if the size > 16MB, you can't embed
- Atomicity of data
 -  if you can not tolerate any inconsistency and want to update at the same time

##One to Many
City: Person
```javascript
people 
{
	name: Andrew,
	city: {
		name: "NY",
		area: ""
	} //duplicate this data in multiple documents, inconsistency problem 
}
```
So we use **True Linking**
```javascript
people 
{
	name: Andrew,
	city: "NYC",
	...
}
City
{
	_id:"NYC",
	...	
}
```

###One to Few
blog posts: comments
```javascript
posts 
{
	name: "",
	comments: [
		
	]
}
```

##Many to Many
Few to Few (Books: Authors) - **Linking**
```javascript
Books 
{
	_id: 12,
	title: "Gone with the wind",
	authors: [27]
}
Authors
{
	_id: 27,
	author_name: "",
	books: [12, 7, 8]
}
```
If for the performance reason, rather than have the arrays of books, you can just embed the books, but in this way, the books may be duplicate in the author collections (multiple authors)

##Multikey Indexes
Students: Teachers
```javascript
students 
{
	_id: "",
	name: Andrew,
	teachers: [1, 7, 10]
}

teachers
{
	_id:1,
	name:"",
}
```
 To find out all students who have a particular teacher
``` 
db.students.ensureIndex("teachers: 1")
db.students.find("teachers: {$all: [0,1]}")
```

##Benefits of Embedding
- Improved Read Performance
 - spinning disk has high latency to get the first byte, but then each additional byte comes very quickly (high bandwidth)
- One Round trip to the DB


##Representation Trees
product category hierarchy 
```
{
  _id: 34,
  name : "Snorkeling",
  parent_id: 12,
  ancestors: [12, 35, 90]
}
```

##When to Denormalize
- 1:1 - Embed 
- 1: Many  - Embed (fromt he many to the one)
- many : many - Link