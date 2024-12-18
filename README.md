# mongoose-fill-promises

mongoose-fill-promises is [mongoose.js](http://mongoosejs.com/) add-on that adds simple api for virtual async fields.

## why?

This just gives you a simple and easy to use API for: 

- implementing db joins without keeping refs to related objects (with refs  you can use `populate`)
- joining mongoose object with any kind of data (with any async service - other db or web service)

## version support - breaking changes

Version 2.0.0 of this library is written as an ESM, and works with mongoose 7.0.0 and above, and requires node 20.0.0 or above.

For compatibility with mongoose < 7 and node < 20 use version 1.x.

This is a fork of `mongoose-fill` which has been archived. Mongoose 7 dropped support for the callback api on queries so this version was modified to use promises.

## api use cases - learn by example

basic use case fills single filed

```javascript
// import of 'mongoose-fill-promises' patches mongoose and returns mongoose object, so you can do:
var mongoose = require('mongoose-fill-promises')
 ...
// note you should set virtual properties options 
// on you schema to get it mongoose-fill-promises work
var myParentSchema = new Schema({
    ....
}, {
  toObject: {
    virtuals: true
  },
  toJSON: {
    virtuals: true 
  }
}

myParentSchema.fill('children', function(callback){
    this.db.model('Child')
        .find({parent: this.id})
        .select('name age')
        .order('-age')
        .exec(callback)
})
...
Parent.findById(1).fill('children').exec().then(function(parent){
    //parent.children <- will be array of children with fields name and age ordered by age
})
```

filling property using single query for multiple objects

```javascript

myParentSchema.fill('childrenCount').value(function(callback){
    // `this` is current (found) instance
    this.db.model('Child')
        .count({parent: this.id})
        .exec(callback)
    // multi is used for queries with multiple fields
    }).multi(function(docs, ids, callback){     
    // query childrenCount for all found parents with single db query
    this.db.model('Child')
        .aggregate([{
            $group: {
                _id: '$parent',
                childrenCount: {$sum: 1}
            }},
            {$match: {'_id': {$in: ids}}}
        ], callback)
})
...

// you can place property name that should be filled in `select` option
Parent.find({}).select('name childrenCount').exec().then(function(parents){
    //parent.childrenCount <- will contain count of children
})
```

using fill options with default values

```javascript
myParentSchema.fill('children', function(select, order, callback){
    this.db.model('Child')
        .find({parent: this.id})
        .select(select)
        .order(order)
        .exec(callback)
}).options('', '-age')
...

// fill children with only `name age` properties ordered by `age`
Parent.findById(1).fill('children', 'name age', 'age').exec().then(function(parent){
    //parent.children <- will be array of children with fields name and age ordered by age
})
```

Also check the code of test for more use cases

## how does it work

- adds fill method to mongoose schema object 
- adds `fill` and `filled` prototype methods to mongoose model 
- patches mongoose query exec method extending query api with `fill` method
- implemented using mongoose virtual setters/getters (actual value is stored in `__propName` property)   


### Installation

npm install mongoose-fill-promises


### Run tests

npm test
