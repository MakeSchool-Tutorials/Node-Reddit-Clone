---
title: "Create a Post"
slug: create-a-post
---

We'll follow this little technical planning list we made, and we can even make sub tasks for each major feature.

1. **Create a post**
  1. Make a posts#new route (`/posts/new`) and template (`posts-new.handlebars`)
  1. Add form to `posts-new` template
  1. Make `create` posts route and check that form data is sending to new route
  1. Add `Post` model with mongoose
  1. Confirm posts are saving to database
1. Show all posts
1. Show one post
1. Comment on posts
1. Create subreddits
1. Sign up and Login
1. Associate posts and comments with their author
1. Make comments on comments
1. Vote a post up
1. Vote a comment up
1. Sort posts by # of votes

# New Post Form

To create a new instance of a resource, we first have to make a form. We'll follow RESTful routing and make the url match this pattern: `/<<RESOURCE NAME PLURAL>>/new`. So in the case of a resource `Post`, the path will be `/posts/new`.

Create this new route and have it render a new template `posts-new.handlebars`.

Now use the [bootstrap form classes](http://getbootstrap.com/css/#forms) to add a form for an object with a `title`, `url`, and `summary` attributes. Your form should have an action that points to a `create` route => `/posts`.

**Remember** to put this form in the center 4 columns of a grid.

```html
<div class="row">
  <div class="col-sm-4"
<form action="/posts">
  <legend>New Post</legend>
  <div class="form-group">
    <label for="post-title">Title</label>
    <input type="title" class="form-control" id="post-title" placeholder="Title">
  </div>
  ...

</form>
```

# Submit the Form

Ok so what happens when you submit this form? No POST `/posts` route! Let's make it.

First make a new folder called `controllers` and in there put the file `posts.js`.

```js
module.exports = function(app) {

  // CREATE
  app.post('/posts', function(req,res) {
    console.log(req.body)
  });

};
```

Now require this file in your `server.js` file and pass in the `app` variable as an argument.

```js
  require('./controllers/posts.js')(app);
```

Now what happens when you submit the form. Is `req.body` defined? No? that's because you need to add the [body parser](https://www.npmjs.com/package/body-parser) module. Add that and get `req.body` to reflect your form inputs.

# Saving to the Database

In order to interact with the MongoDB database we're going to use the npm module [`mongoose`](https://www.npmjs.com/package/mongoose). Mongoose is the ODM - the Object Document Mapper. That means that it maps JavaScript objects in our application to documents in the database. The way Mongoose works is through schemas written in code called Models.

Create the folder `models` and inside put the `post.js` file. Here's a sample model for our `Post` resource.

```js
var mongoose = require('mongoose'),
    Schema = mongoose.Schema;

var PostSchema = new Schema({
  title             : { type: String, required: true }
  , url             : { type: String, required: true }
  , summary         : { type: String, required: true }
});

module.exports = mongoose.model('Post', PostSchema);
```

Now that we have a model, we can require it at the top of our posts controller (`controllers/posts.js`):

```js
var Post = require('../models/post');
```

And put it to use in our create posts endpoint:

```js
var Post = require('../models/post');

module.exports = function(app) {

  // CREATE
  app.post('/posts', function (req, res) {
    // INSTANTIATE INSTANCE OF POST MODEL
    var post = new Post(req.body);

    // SAVE INSTANCE OF POST MODEL TO DB
    post.save(function (err, post) {
      // RESPOND WITH NEW POST
      return res.send({ post: post });
    })
  });

};
```

# Confirming Posts are Saving

So we can save to the database but how can we be sure? There are a couple of ways, we could go into the mongo shell and inspect our database and see that the posts collection has documents in it. Or we can use a program called robomongo to graphically see our database and see what collections and documents we've created.

Use either the mongo shell or robomongo to confirm you are creating posts.

# STRETCH: Adding Created At and Updated At attributes

Let's

```js

var mongoose = require('mongoose'),
    Schema = mongoose.Schema;

var PostSchema = new Schema({
  createdAt       : { type: Date }
  , updatedAt       : { type: Date }

  , title           : { type: String, required: true }
  , url             :
  , summary         : { type: String, required: true }
});

PostSchema.pre('save', function(next){
  // SET createdAt AND updatedAt
  var now = new Date();
  this.updatedAt = now;
  if ( !this.createdAt ) {
    this.createdAt = now;
  }

  next();
});

module.exports = mongoose.model('Post', PostSchema);

```
