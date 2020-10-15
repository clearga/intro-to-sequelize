# Getting started with Sequelize

## Objectives
* Identify an ORM
* Weigh pros and cons between ORMs and raw SQL queries
* Use the Sequelize CLI to create and migrate models
* Utilize Sequelize to create, find, and delete database records
* Explain the purpose of promises and their use in Sequelize

We learned the basics about relational databases (PostgreSQL) and used SQL to query from databases. Now, we're going to incorporate a relational databse into an Express application. But first, some terminology.

## Key terms + definitions

#### ORM

An ORM (Object Relational Mapper) is a piece/layer of software that creates a map between our database into javascript objects that represent our data. This means we can just use JavaScript to create and work with our data instead of writing raw SQL queries.

You can read some more about the benefits of using an ORM [here](http://stackoverflow.com/questions/1279613/what-is-an-orm-and-where-can-i-learn-more-about-it)

**Exercise:** What are 3 pros of an ORM? What are 3 cons?

#### Sequelize

From the Sequelize docs "To put it in a nutshell, it's an ORM (Object-Relational-Mapper). The library is written entirely in JavaScript and can be used in the Node.JS environment." In other words, Sequelize is an ORM that works with relational databases and Node.js. It allows us to do many things including:

- Represent models and their data.
- Represent associations between these models.
- Validate data before they gets persisted to the database.
- Perform database operations in an object-oriented fashion.

#### Model

A model is a javascript object that maps to a data relation (table). You can think of a model as the blueprint for what each row of data is going to contain. Each instance of your model represents one row of data. The ORM allows you perform CRUD on instances of your models which will then map to the equivalent changes in your database.

#### Migration

Migrations (also known as 'schema evolution' or 'mutations') are a way of changing your database schema from one version into another. A migration lays out a plan to change your _model_. When the migration is _run_, it changes your model _and_ uses your ORM to map those changes into your database so your database is alterred accordingly.

# Setup

## Setup part 1 - getting the sequelize-cli tool (you only have to do this once)

We need install a generator so we can use sequelize. Much like our other terminal apps, we will only install this once.

```
npm install -g sequelize-cli
```


### Setup part 2 - starting a new node project

Let's build our first app using Sequelize! First we need to create a node app and include our dependencies. **All in terminal**:

Create a new folder and add an index.js and .gitignore and initialize the repository

```
mkdir userapp
cd userapp
npm init
touch index.js
echo "node_modules" >> .gitignore
```

Add/save dependencies (sequelize needs pg for Postgres)

```
npm install express ejs pg sequelize
```

Create a database and initialize a sequelize project

```
createdb userapp
sequelize init
```

#### For your historical reference...

**WARNING (2017) Edited (2018):**
At one point, sequelize-cli, sequelize, and pg modules were not playing nicely with each other. Luckily, this issue (for version Sequelize 4) has been resolved and we can resume using the current versions of both. In the future, be mindful that many modules you use are maintained by individual third parties and issues like this may come up! 

If you used to use Sequelize 3, keep in mind that Sequelize 4 has breaking changes! If you need to upgrade your app, refer to these [docs](http://docs.sequelizejs.com/manual/tutorial/upgrade-to-v4.html#breaking-changes), which guide you in the update process.

### Setup part 3 - config.json, models and migrations:

In sublime we should now see a bunch of new folders. We now have config, migrations and models. This was created for us when we ran `sequelize init`.

Let's start in the config folder and open up the config.json file. This file contains information about the database we are using as well as how to connect.

We have three settings, one for development (what we will use now), test (for testing our code), and production (when we deploy our app on AWS/Heroku).

Let's change the config.json so it looks like this.

**config/config.json**

```js
{
  "development": {
    "username": "< your postgres username here >",
    "password": "< your postgres user's password >",
    "database": "userapp",
    "host": "127.0.0.1",
    "dialect": "postgres"
  },
  "test": {
    "username": "< your postgres username here >",
    "password": "< your postgres user's password >",
    "database": "userapp_test",
    "host": "127.0.0.1",
    "dialect": "postgres"
  },
  "production": {
    "username": "< your postgres username here >",
    "password": "< your postgres user's password >",
    "database": "userapp_production",
    "host": "127.0.0.1",
    "dialect": "postgres"
  }
}
```

> Note: Remove the angle brackets from the username and password sections

The only thing we are actually changing for database setup, is the **database name**. If you have a username and password for your Postgres server, you'd include those as well.

When we deploy to Heroku (a hosting service for full stack apps), they will provide us a long url that contains password and login that will be secure when deployed. More on this later.

Once this is complete, let's move to the models folder.

## Creating a model and a matching migration

In order to create a model, we start with `sequelize model:create` and then specify the name of the model using the `--name` flag. Make sure your models are **always** singular (table name in plural, model name in singular). After passing in the `--name` flag followed by the name of your model, you can then add an `--attributes` flag and pass in data about your model. Generating the model also generates a corresponding migration. You only need to do this once for your model.

```bash
sequelize model:create --name user --attributes firstName:string,lastName:string,age:integer,email:string
```

> Make sure you do **not** have any spaces between each of the attributes and their data types. Convention matters!

If you want to make changes to your model after generating it - all you have to do is make a change and save it before running the migrate command.

This will generate the following migration

**migrations/\*-create-user.js**

```js
'use strict';
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      firstName: {
        type: Sequelize.STRING
      },
      lasName: {
        type: Sequelize.STRING
      },
      age: {
        type: Sequelize.INTEGER
      },
      email: {
        type: Sequelize.STRING
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('users');
  }
};
```

And a corresponding model:

**models/user.js**
```js
'use strict';
const {
  Model
} = require('sequelize');
module.exports = (sequelize, DataTypes) => {
  class user extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate(models) {
      // define association here
    }
  };
  user.init({
    firstName: DataTypes.STRING,
    lasName: DataTypes.STRING,
    age: DataTypes.INTEGER,
    email: DataTypes.STRING
  }, {
    sequelize,
    modelName: 'user',
  });
  return user;
};
```

## What is this "associate" thing in my model?

In this function, we specify any relations/associations (one to one, one to many or many to many) between our models (hasMany or belongsTo). We'll discuss this more, but always remember, the association goes in the model and the foreign keys go in the migration.

## Running the migration

To run the migration, use the following command.

```
sequelize db:migrate
```

> If you get an error after running the command above, look at your config.json file and search for typos, making sure the database name, your postgres username, and password are all spelled correctly.

If you need to undo the last migration, this command will undo the last migration that was applied to the database.

```
sequelize db:migrate:undo
```

## Using your Models inside an app

Just like using express, body-parser, and the other modules, your models must be required
in order to access them in your app.

```js
var db = require("./models");
db.user.create({
  firstName: 'Brian',
  lastName: 'Hague',
  age: 99
}).then(function(data) {
  // you can now access the newly created task via the variable data
});
```

### CRUD with Sequelize (Using our User model)

#### Create

```js
db.user.create({
  firstName: 'Brian',
  lastName: 'Hague',
  age: 99
}).then(function(data) {
  // you can now access the newly created task via the variable data
});
```

#### Read One

```js
db.user.findOne({
  where: {id: 2}
}).then(function(user) {
  // user will be an instance of User and stores the content of the table entry with id 2. if such an entry is not defined you will get null
});
```

#### Find or Create

The method `findOrCreate` can be used to check if a certain element is already existing in the database. If that is the case the method will result in a respective instance. If the element does not yet exist, it will be created with the provided attributes (a combination of `where` and `defaults`).

In current versions of Sequelize, the library has better support for JavaScript promises and uses them to return results from the database asynchronously. Promised are a part of the language that we're introducing today but will go over in more depth during the React unit. Here is an example of `findOrCreate` in action:

```js
db.user.findOrCreate({
  where: {
    firstName: 'Brian',
    lastName: 'Smith'
  },
  defaults: { age: 88 }
}).then(function([user, created]) {
  console.log(user); // returns info about the user
});
```

We are using the standard `.then()` promise function to run some code once our database returns what we queried for - it's called a promise because our initial request to the database will take some time and so the function "promises" to return a result one war or another. We can get that response inside that promise callback function, where we wrap our two variables in a literal array and pass it as a paramter to the callback function. This will achieve the same effect of providing both the data and the boolean to your promise handler function. 

This will probably be the preferred way of doing this in future versions of Sequelize since the `.spread()` function is actually a feature of another node module called Bluebird. If Sequelize can move away from requiring a 3rd party library, they probably will want to do that so favor the `.then()` way of using it moving forward.

#### Find All

findAll returns more than one instance, which is useful if you need more than one record. find only returns one record.

```js
db.user.findAll().then(function(users) {
  console.log(users);
  // users will be an array of all User instances
});
```

#### Update

```js
db.user.update({
  lastName: 'Taco'
}, {
  where: {
    firstName: 'Brian'
  }
}).then(function(user) {
  // do something when done updating
});
```

#### Delete (destroy)

```js
db.user.destroy({
  where: { firstName: 'Brian' }
}).then(function() {
  // do something when done deleting
});
```

### Promises
After a sequelize statement, we can interact with the return of that object using `.then` and in `findOrCreate` we can use `.spread`.

Finding a user

```js
db.user.findOne({where: {id: 1}});
```

This will execute a statement to find a user, but it will not let us interact with it. Because of the asynchronous nature of a call, we need to use a Promise (a type of callback) to get that data.

```js
db.user.findByPk(1).then(function(foundUser) {
  console.log(foundUser);
  //res.send("myTemplate", {user: foundUser);
});
```

In a `findOrCreate`, a callback will return back an array, instead of a single object. There is a type of callback called `.spread` which will allow us to break apart that array and use similar to a traditional callback.

```js
db.user.findOrCreate({
  where: { firstName: 'Brian' }
}).spread(function(user, created) {
  console.log(user); // returns info about the user
});
```

But as mentioned above, it looks like this syntax will be replaced with more standard promise syntax moving forward. THis would be the new way to use the `findOrCreate` promise:

```js
db.user.findOrCreate({
  where: { firstName: 'Brian' }
}).then(function([user, created]) {
  console.log(user); // returns info about the user
});
```

## Sequelize Promises

The main callback handlers to be used are as follows.

* `.then` - default promise called when a query is completed.
* `.spread` - used to spread an array of values to parameters. This is only used for `findOrCreate` on old versions of Sequelize. This method isn't used in the official documentation but you might it floating around stack overflow posts as little as 1 year old.
* `.catch` - triggered if something goes wrong (an error).
* `.finally` - triggered after all other callbacks. Can be used for cleanup.

The important thing to remember is that all queries take time and are asynchronous, so you MUST use promises to execute code that needs to happen after the query is completed. You will usually use `then`, except possibly for `findOrCreate` (but only if you need to support old versions.)

## Useful Sequelize Documentation Links
* Models
  * [Data types](http://docs.sequelizejs.com/en/latest/docs/models-definition/#data-types)
  * [Validations](http://docs.sequelizejs.com/en/latest/docs/models-definition/#validations)
  * Note that Sequelize **expects the model to be singular**, and uses a pluralizer module to determine what the pluralized model should be. If you're unsure what the pluralizer will output, use this handy app to see what the pluralized model will be. [http://plural-test.herokuapp.com/](Plural Test)
* Querying
  * [Query usage (comparable to SELECT, INSERT, COUNT, MAX, etc.)](http://docs.sequelizejs.com/en/latest/docs/models-usage/)
  * [Destroying records (comparable to DELETE)](http://docs.sequelizejs.com/en/latest/docs/instances/#destroying-deleting-persistent-instances)
  * [Attribute selection](http://docs.sequelizejs.com/en/latest/docs/querying/#attributes)
  * [Querying Basics and Operators (comparable to AND/OR/LIKE/IN)](http://docs.sequelizejs.com/en/latest/docs/querying/#basics)
  * [Pagination and Limiting (comparable to OFFSET, LIMIT)](http://docs.sequelizejs.com/en/latest/docs/querying/#pagination-limiting)
  * [Ordering (comparable to ORDER BY)](http://docs.sequelizejs.com/en/latest/docs/querying/#ordering)
* Configuration and Commands
  * [Sequelize CLI and Migrations](http://docs.sequelizejs.com/en/latest/docs/migrations/#the-cli)
