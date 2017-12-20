# mongo-docs

## Installation

```sh
brew install mongodb
mkdir -p /data/db
```

## Start

```sh
mongod --port=12345 --dbpath=./data/db			# start mongodb instance
mongod --fork --logpath /var/log/mongodb.log  	# start as a daemon
```

## Stop

```sh
use admin; db.shutdownServer();					# through mongo shell
mongod --shutdown								# through terminal
```

## Connect to a remote host (using authentication)

```sh
mongo --username <username> --password <password> --host <database-url> --port <port>
```

## Import data directly into the collection

[Sample Dataset](https://raw.githubusercontent.com/mongodb/docs-assets/primer-dataset/primer-dataset.json)

```sh 
mongoimport --db <dbname> --collection <collectionname> --drop --file ~/downloads/primer-dataset.json
```

## Quick reference to shell

```sh
show dbs										# List of all databases
use <db>										# Switch to <db>
show collections								# List of all collections
show users										# List of users
show profile									# Print the five most recent operations that took 1 millisecond or more

db.fromColl.renameCollection(<toColl>)			# Rename collection from fromColl to <toColl>
db.getCollectionNames()							# List of all collections
db.dropDatabase()								# Drops the current database
```


## Insert

### Single document

```sh
db.inventory.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)

db.inventory.insert(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)

db.products.save( 
	{ item: "book", qty: 40 } 
)

```

### Multiple documents

```sh
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])

db.inventory.insert([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])

db.products.save([ 
	{ item: "book", qty: 40 }, 
	{ item: "pencils", qty: 10 }
])
```

## Query

```sh
# Top level
db.restaurants.find( { "borough": "Manhattan" } )		

# Nested
db.restaurants.find( { "address.zipcode": "10075" } )	

# Field in an array
db.restaurants.find( { "grades.grade": "B" } )			

# Here, size field needs to match exact `{ h: 14, w: 21, uom: "cm" }` in specified order
db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } )

# Here, tags field needs to match exact `["red", "blank"]` in specified order
db.inventory.find( { tags: ["red", "blank"] } )

# Find an array that contains both the elements "red" and "blank" without any order
db.inventory.find( { tags: { $all: ["red", "blank"] } } )

# Array index i.e. 2nd element
db.inventory.find( { "ages.1": { $gt: 30, $lt: 70 } } )

# Array length
db.inventory.find( { "tags": { $size: 3 } } )

# Array index and nested combined
db.inventory.find( { 'instock.0.qty': { $lte: 20 } } )

# Projection
db.inventory.find( { status: "A" }, { item: 1, status: 1, _id: 0, "instock.qty": 1 } )

# Item field is neither null or doesn't exist
db.inventory.find( { item: null } )
db.inventory.find( { item : { $exists: false } } )
```

## Update

```sh

# Top level
db.restaurants.update(
    { "name" : "Juni" },
    { 
      $set: { "cuisine": "American (New)" },
    }
)

# Nested
db.restaurants.update(
  { "restaurant_id" : "41156888" },
  { 
  	$set: { "address.street": "East 31st Street" } 
  }
)

# Multiple update
db.restaurants.update(
  { "address.zipcode": "10016", cuisine: "Other" },
  {
    $set: { 
    	cuisine: "Category To Be Determined" 
    },
  },
  { multi: true }
)	

# Replace the entire matched document
db.restaurants.update(
   { "restaurant_id" : "41704620" },
   {
     "name" : "Vella 2",
     "address" : {
              "coord" : [ -73.9557413, 40.7720266 ],
              "building" : "1480",
              "street" : "2 Avenue",
              "zipcode" : "10075"
     }
   }
)
```

## Delete

```sh

# Remove all
db.restaurants.remove( { "borough": "Manhattan" } )

# Just one
db.restaurants.remove( { "borough": "Queens" }, { justOne: true } )

# Cleans up the collection
db.restaurants.remove( {} )
```


## Operators

```sh
	
db.restaurants.find( { "grades.score": { $gt: 30 } } )	

db.inventory.find( { status: { $in: [ "A", "D" ] } } )

db.inventory.find( { qty: { $nin: [ 5, 15 ] } } )

db.inventory.find( { "carrier.state": { $ne: "NY" } } )

# Logical AND
db.restaurants.find( { "cuisine": "Italian", "address.zipcode": "10075" } )	

db.inventory.find( {
    $and : [
        { $or : [ { price : 0.99 }, { price : 1.99 } ] },
        { $or : [ { sale : true }, { qty : { $lt : 20 } } ] }
    ]
} )

# Logical OR
db.restaurants.find( {
   $or: [ 
   		{ "cuisine": "Italian" }, { "address.zipcode": "10075" } 
   	] 
} )

# Expression: where spent field exceeds budget field
db.monthlyBudget.find( { $expr: { $gt: [ "$spent" , "$budget" ] } } )

# $where using javascript expressions
db.monthlyBudget.find( { $where: "this.spent > this.budget" } );
db.monthlyBudget.find( { $where: function() { return this.spent > this.budget } } );

# $elemMatch: matches documents that contain an array field with at least one element that matches all the specified query criteria
# { _id: 1, results: [ 82, 85, 88 ] }
# { _id: 2, results: [ 75, 88, 89 ] }

db.scores.find( {
   results: { 
   		$elemMatch: { $gte: 80, $lt: 85 } 
   }
} )

# For data:
# { _id: 1, results: [ { product: "abc", score: 10 }, { product: "xyz", score: 5 } ] }
# { _id: 2, results: [ { product: "abc", score: 8 }, { product: "xyz", score: 7 } ] }
# { _id: 3, results: [ { product: "abc", score: 7 }, { product: "xyz", score: 8 } ] }

db.scores.find({ $and: [ { "results.product": "abc" },  { "results.score": { $eq: 7 } }  ] })

# produces
# { "_id" : 2, "results" : [ { "product" : "abc", "score" : 8 }, { "product" : "xyz", "score" : 7 } ] }
# { "_id" : 3, "results" : [ { "product" : "abc", "score" : 7 }, { "product" : "xyz", "score" : 8 } ] }

#while

db.products.find( { 
	results: {
		$elemMatch: { product: "abc", score: { $eq: 7 } }
	} 
} )

# produces
# { "_id" : 3, "results" : [ { "product" : "abc", "score" : 7 }, { "product" : "xyz", "score" : 8 } ] }

# The results differ because `$elemMatch` works specifically for an array while a normal query is for any type.
```

## Bulk Write 

```sh
db.characters.bulkWrite([
    { 
     	insertOne: {
           "document": {
           		"name": "Avengers", "year": 2010, "language": "en"
           }
        }
    },
    { 
     	insertOne: {
           "document": {
              "name": "Hurt Locker", "year": 2008, "language": "en"
           }
        }
    },
    { 
     	updateOne: {
           "filter" : { "name" : "Avengers" },
           "update" : { $set : { "year" : 2017 } }
        }
    },
    { 
     	deleteOne: { 
     		"filter" : { "year" : 2008 } 
     	}
    }
]);
```


## Data Aggregation

## Indexes