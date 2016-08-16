# Lint
## Library for atomic-ish operations in MongoDB

Lint allows you to carry out edits on your database as a series of steps. If an error occurs on one of the steps, the database is returned to it's initial state (it's state before the transaction started). 

## Getting Started:

Install [node.js](https://nodejs.org) and [mongoDB](https://www.mongodb.com/download-center)

Start mongoDB in a terminal: ```mongod```

Then:
```npm install oj-lint```

## Usage:
```javascript
var Lint = require("oj-lint");
```

### Lint.init(db, _collection, options): Initialize Lint
> db (required): [mongoose](https://github.com/Automattic/mongoose) instance or [connection string](https://docs.mongodb.com/manual/reference/connection-string/)

> _collection (optional): name of collection to be used internally by Lint

> options (optional. lol): Connection options. Same as [mongoose connection options](http://mongoosejs.com/docs/connections.html#options)

<br>If you're using mongoose in your project initialize Lint with mongoose:

```javascript
var mongoose = require("mongoose");

mongoose.connect("mongodb://127.0.0.1:27017/testDB");

//remember, _collection is optional
Lint.init(mongoose, "lint_collection_name_if_you_want_to_specify");
```

Without mongoose, Initialze Lint like so:

```javascript
//options object (http://mongoosejs.com/docs/connections.html#options)
var options = {
  user: "teh_huose_kat"
  pass: "teh_Kitti_passwrod"
}

var collection = "lint_collection_name_if_you_want_to_specify";

//remember, _collection and options are optional
Lint.init("mongodb://127.0.0.1:27017/testDB", collection || null, options || null);
```
<br>
### Lint.Task(): Create a Lint task

After intitializing Lint, create a task like so:

```javascript
var task = Lint.Task();
```
<br>
### task.initModel(modelName, schema): To initialize a model with a Schema.
  > modelName (required): Name of the collection associated with this model
  
  > schema (required): Same as object passed to [mongoose Schema](http://mongoosejs.com/docs/guide.html#definition). Also see [validation](http://mongoosejs.com/docs/validation.html)
  
  <br>If you're using mongoose, define your models with mongoose wherever possible. If the model has been defined by mongoose before this function is called, mongoose will throw an OverwriteModelError and if it was defined by Lint, Lint will throw an Error. Models can be defined only once.
  
  ```javascript
  var schema = {
    name: {type: String, required: true}
    , specials: [{title: String, year: Number}]
  }
  
  task.initModel("comedians", schema);
  ```
  
  Save operations to the "comedians" model will validate against the schema;

<br>
### task.save(model, doc): To save a document</b>
  > model (required): Name of the collection we're saving to or a mongoose model or a mongoose document

  > doc (optional): object to save or a mongoose document
	
  <br>these are all valid:
  
  ```javascript
  var Cars = mongoose.model("cars", new Schema({make: String, year: Number}));
  var toyota = new Cars({make: "Toyota", year: 2015});

  task.save("cars", {make: "Toyota", year: 2015})
  task.save(Cars, {make: "Toyota", year: 2015})
  task.save("cars", toyota)
  task.save(Cars, toyota)
  task.save(toyota)
  ```
  
  *Note: No changes will be made to to your database until you call task.run()*
  
<br> 
[mongoose update]: <http://mongoosejs.com/docs/api.html#model_Model.update> 
[mongodb]: <https://docs.mongodb.com/manual/core/document/#document-query-filter>
### task.update(model, condition, data): To update a document
  > model (required): Name of the collection we're saving to or a mongoose model or a mongoose document

  > condition (required): same as in [mongoose update][] and [mongodb][]
  
  > data (optional): data to update with same as in [mongoose update][] and [mongodb](https://docs.mongodb.com/manual/reference/method/db.collection.update/#update-parameter)
  
  <br> These are all valid
 
  ```javascript
  var Cars = mongoose.model("cars", new Schema({make: String, year: Number}));
 
  // update the value of year on all cars with make === "Toyota" to 2016
  task.update("cars", {make: "Toyota"}, {year: 2016});
  task.update(Cars, {make: "Toyota"}, {year: 2016});
  
  Cars.findOne({make: "Toyota"}, function(toyota){
    //update just this toyota
    task.update(toyota, {year: 2016});
  });
 ```
 
  *Note: No changes will be made to to your database until you call task.run()*
