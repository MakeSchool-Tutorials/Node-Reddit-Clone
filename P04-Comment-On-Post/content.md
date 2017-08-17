---
title: "Comment on Posts"
slug: comment-on-post
---

Alright next step! We can see those posts, let's comment on them.

1. Create a post
1. Show all posts
1. Show one post
1. **Comment on posts**
  1. Make a new comment form in the posts#show template
  1. Make a create route for comments
  1. Associate comments with posts
  1. Display comments
1. Create subreddits
1. Sign up and Login
1. Associate posts and comments with their author
1. Make comments on comments
1. Vote a post up
1. Vote a comment up
1. Sort posts by # of votes


# New Comment Form (with Nested Route)

Remember, we are always coding using an agile and user-centric approach, so we are going to always start whatever feature from what the user sees and does. Then we'll code back towards what the server and code does.

So, if we want to allow users to comment on these posts, first we can add a comment form to our posts#show page. This form will send its data to the a path that resolves to the comments#create action. The path for this link will follow the standard nested RESTful convention `/<<PARENT RESOURCE PLURAL>>/<<PARENT ID>>/<<CHILD RESOURCE PLURAL>>`.

Let's use a textarea for the comment attribute `body`.

```html
...
  <form action="/posts/{{post._id}}/comments">
    <textarea class='form-control' name="body" placeholder="Comment"></textarea>
    <div class="text-right">
      <button type="submit">Save</button>
    </div>
  </form>
```

If you submit the form, it fails because there is no POST route to `/posts/{{post._id}}/comments` yet. Let's fix that working from the outside (what the user sees and does) into our server.

# Make Create Comment Route

Now we need a create comment route. We can start with the code we used for the create route for posts.

> [info]
> Remember that a route can be called a number of different names: an endpoint, a webhook, a path, and others.

Follow the pattern you used for the Post resource to create a Comment resource.

1. Create a comments controller in a new file `comments.js` in your `controllers` folder

  ```js
  module.exports = function(app) {

  };
  ```

1. Export the comments controller into the `server.js`

  ```js
    require('./controllers/posts.js')(app);
  ```

1. Make the CREATE in a nested route (hint: `/posts/:postId/comments`)

  ```js
  // CREATE
  app.post('/posts/:postId/comments', function (req, res) {
    // INSTANTIATE INSTANCE OF MODEL
    var comment = new Comment(req.body);

    // SAVE INSTANCE OF POST MODEL TO DB
    comment.save(function (err, comment) {
      // REDIRECT TO THE ROOT
      return res.redirect(`/`);
    })
  });

  ```

1. Create a Comment model in a `comment.js` file

  ```js
  var mongoose = require('mongoose'),
      Schema = mongoose.Schema;

  var CommentSchema = new Schema({
    content             : { type: String, required: true }
  });

  module.exports = mongoose.model('Comment', CommentSchema);
  ```

1. Require the comment model in the comments controller

  ```js
  var Comment = require('../models/comment');
  ```

1. Create a comment by submitting your form
1. Confirm that comments are saving by inspecting the database

# Associating Comments and Posts

**The Gotcha** - the gotcha here is that if you just use the same code from `posts.js` you will only create comments in their own collection and not associate them with their parent post. We're going to use a **Reference Association** meaning we will reference the child document by its id in the parent's document. The child's id acts similarly to a **Foreign Key** in a SQL database. So let's do it.

First we can do the controller logic, then model logic.

In the controller we need find the parent Post from the `:postId` we have in the url parameters, then associate this parent with the comment by pushing the comment into an array in the parent's attribute `comments` that we haven't created yet.

```js
// CREATE
app.post('/posts/:postId/comments', function (req, res) {
  var comment = new Comment(req.body);

  Post.findById(req.params.postId).exec(function (err, post) {
    comment.save(function (err, comment) {
      post.comments.unshift(comment);
      post.save();

      return res.redirect(`/posts/` + post._id);
    })
  })
});
```

Why did I recommend we use unshift here instead of push?

> [solution]
> `unshift` adds an element to the front of an array, while `push` adds it to the end. Reddit puts its newest comments at the top, so we want the default order to be reverse chronological order.

Next we need to add an array attribute to the mongoose `Post` model.

```js
  , comments       : [{ type: Schema.Types.ObjectId, ref: 'Comment' }]
```

Finally, create some new comments and confirm that their `_id`'s are being added to this `comments` attribute.

# Displaying Comments

Now that we have the comments associate we can see them in the parent `post` object. Let's add them to the posts#show template below the new comment form.

```html
{{#each post.comments}}   
  {{this}}
{{/each}}
```

What do you see?

Just the id's right? When we do a reference association, we only save the id's into the parent's document. In order to replace these id's with the actual child document, we have to use the mongoose function `.populate()` when we fetch the parent from the database. Like this:

```js
Post.findById(req.params.id).populate('comments').exec(function (err, post) {
  res.render('posts-show', { post: post });
});
```

Now do we see the comments?

Just one more change, you have to access the `content` attribute of each comment. You can add more style to these if you like. Perhaps a paragraph tag to start.

```html
{{#each post.comments}}   
  <p>{{this.content}}</p>
{{/each}}
```

Onward!
