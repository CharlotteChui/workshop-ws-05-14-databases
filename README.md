# CS52 Workshops: Databases!

![](https://media.giphy.com/media/xT5LMC67mF1yMpE6HK/giphy.gif)

*Insert something here to hype em up about databases*

## Overview
In this workshop, we're going to rebuild our SA7 assignment. In case you don't remember, that's the lab with polls and upvotes/downvotes. Instead of using MongoDB and Mongoose, we're going to use PostgreSQL and Sequelize! You will learn how to install PostgreSQL and work with it through the command line interface. Then, we'll use Sequelize, a promise-based ORM (Object Relation Mapping) for Node.js. 

## Setup
### What the hell is PostgreSQL?
We know our presentation about databases was interesting, and you probably know all about PostgreSQL now. But just in case you need a reminder, PostgreSQL (if you wanna impress your friends, call it Postgres) is an open source relational database management system. 

### Installing PostgresSQL
Open up your Terminal and install `postgresql` with `brew`. 

`brew install postgresql`

Wow, you're killing it. You just installed PostgreSQL. Now, let's get it running. You can do this with the following command:

`brew services start postgresql`

Your terminal should have spit out `==> Successfully started `postgresql` (label: homebrew.mxcl.postgresql)`.

Alright, you CS GOD, now that PostgreSQL is running, let's connect to it.

## Step By Step
## PostgreSQL Command Prompt

`psql` is the PostgreSQL interactive terminal. Running `psql` will connect you to a  PostgreSQL host. Just in case you want use PostgreSQL on your final project, here are some options for connecting with `psql`:

```
-h — --host=HOSTNAME | database server host or socket directory (default: "local socket")
-p — --port=PORT | database server port (default: "5432")
-U — --username=USERNAME | database username (default: "your_username")
-w — --no-password | never prompt for password
-W — --password | force password prompt (should happen automatically
```

We're not trying to do anything fancy, so let's connect to the default `postgres` database with the following command:

`psql postgres`

You're now inside `psql` in the `postgres` database. The prompt ends in `#` which tells us we're logged in as the superuser, or root. Commands within `psql` start with a backlash. To see what database, user, and port we've connected to, run `\conninfo`. 

Here are some other commands:

```
\q | Exit psql connection
\c | Connect to a new database
\dt | List all tables
\du | List all roles
\list | List databases
```

## Create a User and Log in
We're gonna create a role called `me` (or whatever you wish) and give it a password `password` (or whatever you wish). According to the PostgreSQL documentation, "a role is an entity that can own database objects and have database privileges; a role can be considered a 'user', a 'group' or both depending on how it is used." In this case, we'll be using a role as a user. Run the following command:

`CREATE ROLE me WITH LOGIN PASSWORD 'password';`

We can use the command `ALTER ROLE` to change the attributes of a PostgreSQL role. We want to alter the role of `me` so that it can create other roles and new databases.

`postgres=# ALTER ROLE me CREATEDB;`

If you now run `\du`, you should see something like this:

```
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 Tim       | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 me        | Create DB
 ```

Now, run `\q` to quit. You should now be in the default Terminal connection.

Run `psql -d postgres -U me` to connect postgres with me. Our prompt should now show `postgres=>` instead of `postgres#`, meaning we're not logged in as a superuser (root).

## Create a Database
Run the following to create your 'Polls' database:

`CREATE DATABASE polls; \list quit`

## Sequelize 
If you haven't already, fork this repo, clone it, and cd into the directory. Run the following commands

```
brew upgrade node
yarn install
yarn add sequelize pg pg-hstore
```

Here, we're adding sequelize, node-postgres (a PostgreSQL client for Node.js), and pg-hstore (a node package for serializing/deserializing JSON data to hstore format).

### Part 1 -- Connecting Sequelize to PostgreSQL
Open up your directory in Visual Studio, Atom, or whatever. (If you're a CS GOD, you'll do the rest of this workshop in emacs). Navigate to `src/models/index.js` 

Underneath the 'PART 1' comment, change 'database', 'username', 'password' to whatever you set in the PG part.

So, you end up with something like this:

`const db = new Sequelize('postgres://me:password@localhost:5432/polls');`

This connects to the PostgreSQL database with a new instance of Sequelize. The address localhost:5432 works because we kept running postgres as a service in brew!

### Part 2 -- Models 

Under `src/models`, create `poll.js` and `author.js`.

#### Part 2.1 -- Author Model
First, we want to create a `author` model. Paste the following in `author.js`:

```
const author = (sequelize, DataTypes) => {
  const Author = sequelize.define('author', {
    name: {
      type: DataTypes.STRING,
      unique: true, 
    },
  });

  Author.associate = (models) => {
    Author.hasMany(models.Poll, { onDelete: 'CASCADE' });
  };

  return Author;
};

export default author;
```
First, we've defined the `author` model. Notice, the name is unique; we can't have different users with the same name. 

Next, we create an association by calling  `hasMany` providing the `poll` model as the first argument. We use `hasMany` because each author can write multiple polls. In creatin associations, you can also use `hasOne`, `belongsTo`, and `belongsToMany`. The documentation can be found here: <http://docs.sequelizejs.com/class/lib/associations/base.js~Association.html>

#### Part 2.2 -- Poll Model

Now, let's create our 'poll' model. Paste the following in `poll.js`:

```
const poll = (sequelize, DataTypes) => {
  const Poll = sequelize.define('poll', {
    text: {
      type: DataTypes.STRING,
    },

    imageURL: {
      type: DataTypes.STRING,
    },

    upvotes: {
      type: DataTypes.INTEGER,
      defaultValue: 0,
    },

    downvotes: {
      type: DataTypes.INTEGER,
      defaultValue: 0,
    },

  }, {
    getterMethods: {
      score() { return this.upvotes - this.downvotes; },
    },
  });

  Poll.associate = (models) => {
    Poll.belongsTo(models.Author);
  };

  return Poll;
};

export default poll;
```

Here, we're doing the same thing we did with the `author` model. We're defining the `poll` model with `text`, `imageURL`, `upvotes`, and `downvotes`. Notice, `upvotes` and `downvotes` have a default value of 0.

Then, we associate the `poll` with an `author` using `associate` and `belongsTo`. 

### Part 3 -- Setting up the API
If you open up <http://localhost:9090/> in your browser, you should see the SA7 assignment. Now, let's fetch all the polls.

Open up `server.js` and add the following to the top:

```
import models, { sequelize } from './models';
import createAuthorsWithPolls from './polls

// sync Sequelize
const eraseDatabaseOnSync = true;
sequelize.sync({ force: eraseDatabaseOnSync }).then(async () => {
  if (eraseDatabaseOnSync) {
    createAuthorsWithPolls();
  }
});
```

#### Part 3.1 -- GET all the posts
This is basically the same as it was when we did SA7 with MongoDB. The syntax is a bit different, and we're adding the author to the model. Beneath the 'PART 3.1' comment, add the following:

```
// default index route
app.get('/', (req, res) => {
  models.Poll.findAll({
    include: [{ model: models.Author }],
  })
    .then((polls) => {
      res.render('index', { polls });
    }).catch((error) => {
      res.send(`error: ${error}`);
    });
});
```

#### Part 3.2 -- POST for new posts
ssignment with some polls we left there for you. 

Now, let's make it so our users can create new posts. Add the following below the 'PART 3.2' comment.

```
app.post('/new', (req, res) => {
  const newpoll = {
    text: req.body.text,
    imageURL: req.body.imageURL,
  };

  models.Poll.create(
    newpoll, {
      include: [{ model: models.Author }],
    },
  )
    .then((poll) => {
      models.Author.findOrCreate({ where: { name: req.body.author } })
        .then((author) => {
          poll.setAuthor(author[0]);
          poll.save();
        })
        .then(() => {
          res.redirect('/');
        });
    });
});
```

Here, we are creating the `poll` object with the information from the form users can fill out. We then 'find or create' the author they provided and link it to the pool. Finally, we redirect to the home page.

#### Part 3.3 -- POST for votes

Now, let's make it so we can upvote/downvote polls. Add the following below the 'PART 3.3' comment.

```
app.post('/vote/:id', (req, res) => {
  const vote = (req.body.vote === 'up');

  models.Poll.findByPk(req.params.id).then((poll) => {
    console.log(`updating vote: ${poll} ${vote}`);
    if (vote) {
      poll.increment('upvotes');
    } else {
      poll.increment('downvotes');
    }
    res.send(poll);
  });
});
```

#### Part 3.4 -- GET author's posts

Now that we've created a relational database associating authors with posts, we can request only the posts of a specific author. Add the following below the `PART 3.4' comment.

```
app.get('/author/:id', (req, res) => {
  models.Poll.findAll({
    where: { authorId: req.params.id },
    include: [{ model: models.Author }],
  })
    .then((polls) => {
      res.render('index', { polls });
    }).catch((error) => {
      res.send(`error: ${error}`);
    });
});
```

Here, we're requesting all the polls associated with a specific author. Pretty cool, right? 

### Add more polls
Navigate into `src/polls.js`. You'll see we've already created some polls for you. 

Notice how `polls` was not a field in the model, but now it became one after the 'associate' part. Sequelize pluralized it from poll to polls by itself! WOWOWOW. 

## Part 4: 

@Mihovil, this is the section for eraseDataOnSync.

## Summary / What you Learned

* [ ] can be checkboxes

## Reflection

*2 questions for the workshop participants to answer (very short answer) when they submit the workshop. These should try to get at something core to the workshop, the what and the why.*

* [ ] 2 reflection questions
* [ ] 2 reflection questions

## Resources
* <https://blog.logrocket.com/setting-up-a-restful-api-with-node-js-and-postgresql-d96d6fc892d8>


