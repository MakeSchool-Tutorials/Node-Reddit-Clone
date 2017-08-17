---
title: "Comments on Comments: Part Deux"
slug: comments-on-comments-part-deux
---

So last lesson was about thinking about and weighing the pros and cons of a few implementations of comments on comments. Now we're going to walk through how to actually implement one strategy.

1. Create a post
1. Show all posts
1. Show one post
1. Comment on posts
1. Create subreddits
1. Sign up and Login
1. Associate posts and comments with their author
1. **Make comments on comments**
  1. Make comments embedded documents instead of reference.
  1. Make comments on posts work.
  1. Make comments embedded inside comments.
  1. Test comments on comments are working.
1. Vote a post up
1. Vote a comment up
1. Sort posts by # of votes

# Embedding Comments in Posts

I hope in your conversations about how to put comments on comments you came to the same conclusion I did, which was to make all comments embedded into posts and all comments on comments embedded into those comments.

```json
{
  _id: "ji2roi3ji23j2j3",
  user: "fu0fa90faa99a0a",
  body: "Awesome site to share",
  url: "https://www.google.com",
  comments: [
    {
      _id: "afk3jk4k23j4h232",
      user: "lk11l2j31l24j1kl",
      content: "great post"
      comments: [
        {
          _id: "afk3jk4k23j4h232",
          user: "lkj4l1j41kl1343",
          content: "great comment",
          comments: []
        }
      ]
    }
  ]
}
```

The reasons for using embedded documents is the reference document strategy would not work to create a tree of comments because you cannot use the method `.populate()` to populate more than one or two layers down the hierarchy, but we want all the comments of comments. We'll see too that as long as we track each embedded comment with its own unique `_id` we'll be able to edit and vote on comments, no problem. Maybe we'll even be able to sort them by number of votes. We'll see.

So the first thing we have to do is swap comments from being reference doc to being an embedded document.

# Switch Comments From Reference to Embedded

We always think about what the user will see first, and change that. But what the user see's won't change at all, so our first step is to change our controller logic, and then we'll update our models.

The [mongoose docs](http://mongoosejs.com/docs/2.7.x/docs/embedded-documents.html) on the topic show us that with embedded documents, we are really just updating the parent document and then saving that. So let's update our comments#create route to look like this:

```js
// CREATE
app.post('/posts/:postId/comments', function (req, res) {
  // FIND THE PARENT POST
  Post.findById(req.params.postId).exec(function (err, post) {
    // UNSHIFT A NEW COMMENT
    post.comments.unshift(req.body);
    // SAVE THE PARENT
    post.save();

    // REDIRECT BACK TO THE PARENT POST#SHOW PAGE TO SEE OUR NEW COMMENT IS CREATE
    return res.redirect(`/posts/` + post._id);
  })
});
```

Now we need the model to expect this embedded doc.

Mongoose allows us to represent embedded documents by embedding the schema object into the schema of the parent. Like this:


```js
var Comments = new Schema({
    title     : String
  , body      : String
  , date      : Date
});

var BlogPost = new Schema({
    author    : ObjectId
  , title     : String
  , body      : String
  , date      : Date
  , comments  : [Comments]
  , meta      : {
        votes : Number
      , favs  : Number
    }
});

mongoose.model('BlogPost', BlogPost);
```

So for our purposes we need to get that CommentsSchema into our Post model. We can either require the Comment model into our `post.js` field. Or we can just move the code right into it.

```js
var mongoose = require('mongoose'),
    Schema = mongoose.Schema;
    Comment = require('comment')

var PostSchema = new Schema({
  title             : { type: String, required: true }
  , url             : { type: String, required: true }
  , summary         : { type: String, required: true }
  , comments        : [Comment.schema]
});
```

Now that the model is setup to be embedded, see if you can create comments.

Do you still need to use the method `.populate()` to see the comments of a post? Better remove that!

# Embedding Comments Into Comments

Now we can make it so a user can comment on a comment to create our nice hierarchical tree of nested comments.

Let's again use the user's experience as a starting point. So a user who wants to make a comment on a comment, can click a "reply" link that opens up a form that displays that specific comment and a textarea and a Reply button.

```html
  ...
  <a href="/posts/{{post._id}}/comments/{{comment._id}}/replies/new">Reply</a>
  ...
```

Let's make a new `replies.js` file in our `controllers` folder. In there we're going to need access to both the Post and Comment models and we'll end up with two route NEW and CREATE.

```js
var Post = require('../models/post');
var Comment = require('../models/comment');
var User = require('../models/user');

module.exports = function(app) {
  // NEW REPLY
  app.get('/posts/:postId/comments/:commentId/replies/new', function(req, res, next) {
    Post.findById(req.params.postId).exec(function (err, post) {
      Comment.findById(req.params.commentId).exec(function (err, comment) {
        res.render('replies-new', { post: post, comment: comment });
      })
    });
  });

  // CREATE REPLY
  app.post('/posts/:postId/comments/:commentId/replies', function(req, res, next) {
    console.log(req.body);
  });

}

```

Now make your `replies-new` template and have the content sit in the middle 6 columns of the 12 column grid:

```html
<div class="row">
  <div class="col-sm-6 col-sm-offset-3">
    <form action="/posts/{{post._id}}/comments/{{comment._id}}/replies">
      <div class="form-group">
        <textarea name="content" class="form-control" id="reply-content" placeholder="Reply"></textarea>
      </div>

      <div class='text-right'>
        <button type="submit" class="btn btn-primary">Reply<button>
      </div>
    </form>
  </div>
</div>
```

Ok now our template and form is there, but what happens when we submit it? Because we already made the create route and left a `console.log(req.body)`, if you submit that form, you should see the console log output the content. Do you see it?

The next step is to write our replies#create route logic.

```js
  // CREATE REPLY
  app.post('/posts/:postId/comments/:commentId/replies', function(req, res, next) {
    // LOOKUP THE PARENT POST
    Post.findById(req.params.postId).exec(function (err, post) {
      // FIND THE CHILD COMMENT
      var comment = post.comments.id(req.params.commentId);
      // ADD THE REPLY
      comment.comments.unshift(req.body);
      // SAVE THE CHANGE TO THE PARENT DOCUMENT
      post.save();

      // REDIRECT TO THE PARENT POST#SHOW ROUTE
      res.redirect('/posts/' + post._id);
    });
  });
```

And lastly we need to update our model so that comments have embedded instances of its self.

```js
var mongoose = require('mongoose'),
    Schema = mongoose.Schema;

var CommentSchema = new Schema({
  content            : { type: String, required: true }
  , comments           : [CommentSchema]
});

module.exports = mongoose.model('Comment', CommentSchema);
```

How strange! And yet how strange is Reddit itself?

Now when you submit the reply form, what occurs? Can you confirm in the database that a nested embedded comment document is created?

Finally, let's set up our `post-show` template to show these sub comments as well once they are created. If we just try to manually write in our comments and their comments, we won't be able to represent the whole tree. We'll need to use a **Partial Template** to make a recursive representation of all the comments and their comments.

```html
<div class="row">
  {{#each post.comments}}   
    {{> partials/comment}}
  {{/each}}
</div>
```

Now let's create a new folder in the `views` folder called `partials` and create a file in there called `comment.handlebars`. And inside that template we'll call itself again, so it loops until every comment is displayed.

```html
<div class="col-xs-12 comment-indent">
  <p>{{this.content}}</p>
  {{#each this.comments}}
    {{> comment}}
  {{/each }}
</div>
```

We can give each comment a bit of an intent by creating the class `.comment-indent`.

```css
.comment-indent {
  margin-left: 10px;
}
```

What other styles would you add?
