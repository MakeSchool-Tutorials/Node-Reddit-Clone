---
title: "Comments on Comments: Part Deux"
slug: comments-on-comments-part-deux
---

So last lesson we were thinking through how to put comments inside other comments. Now we are going to actually do it!

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
1. Sort posts by # of votes

# Embedding Comments in Posts

In your conversations about how to put comments on comments, did you arrive at the following solution? You can make all comments embedded into posts, and all comments on comments embedded _into those comments_.

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

The reasons for using embedded documents is the reference document strategy would not work to create a tree of comments --- you cannot use the `.populate()` method to populate more than one or two layers down the hierarchy. We want all the comments of comments. As long as we track each embedded comment with its own unique `_id`, we'll be easily able to edit and vote on comments. Perhaps we'll even be able to sort them by number of votes --- we'll see.

The first thing we have to do is swap comments from being reference documents to being an embedded document.

# Switch Comments From Reference to Embedded

We always think about what the user will see first, and change that. In this case, what the user sees won't change at all! Our first step is to change our controller logic, and then propagate the change to our models.

The [mongoose docs](http://mongoosejs.com/docs/2.7.x/docs/embedded-documents.html) on the topic describe that with embedded documents, we are really just updating the parent document, then saving that. Consequently, let's update our comments#create route to look like this:

```js
// CREATE
app.post("/posts/:postId/comments", function(req, res) {
  // FIND THE PARENT POST
  Post.findById(req.params.postId).exec(function(err, post) {
    // UNSHIFT A NEW COMMENT
    post.comments.unshift(req.body);
    // SAVE THE PARENT
    post.save();

    // REDIRECT BACK TO THE PARENT POST#SHOW PAGE TO SEE OUR NEW COMMENT IS CREATE
    return res.redirect(`/posts/` + post._id);
  });
});
```

Now, we need the model to expect this embedded doc.

Mongoose allows us to represent embedded documents by embedding the schema object into the parent object's schema. For example:

```js
var Comments = new Schema({
  title: String,
  body: String,
  date: Date
});

var BlogPost = new Schema({
  author: ObjectId,
  title: String,
  body: String,
  date: Date,
  comments: [Comments],
  meta: {
    votes: Number,
    favs: Number
  }
});

mongoose.model("BlogPost", BlogPost);
```

For our purposes, we need to get that `CommentsSchema` into our `Post` model. We can either require the `Comment` model into our `post.js` field. Alternatively, we can simply move the code right into `models/post.js`. The following is an example of the former:

```js
const mongoose = require("mongoose");
const Schema = mongoose.Schema;
const Comment = require("models/comment");

var PostSchema = new Schema({
  title: { type: String, required: true },
  url: { type: String, required: true },
  summary: { type: String, required: true },
  comments: [Comment.schema]
});
```

The model is now set up to be embedded. Test the site and make sure you can create comments. Do you still need to use the method `.populate()` to see the comments of a post? Better remove that!

# Embedding Comments Into Comments

Now we can make it so a user can comment on a comment to create a nice hierarchical tree of nested comments.

Again, utilize the user experience as a starting point. A user who wants to make a comment on a comment can click a "reply" link that opens up a form displaying the specific comment and a textarea, as well as a Reply button.

```html
  ...
  <a href="/posts/{{post._id}}/comments/{{comment._id}}/replies/new">Reply</a>
  ...
```

Let's make a new `replies.js` file in our `controllers` folder. Within, we'll need access to both the `Post` and `Comment` models. At the end, we'll have two freshly implemented routes: `NEW` and `CREATE`.

```js
var Post = require("../models/post");
var Comment = require("../models/comment");
var User = require("../models/user");

module.exports = app => {
  // NEW REPLY
  app.get("/posts/:postId/comments/:commentId/replies/new", (req, res) => {
    let post;
    Post.findById(req.params.postId)
      .then(p => {
        post = p;
        return Comment.findById(req.params.commentId);
      })
      .then(comment => {
        res.render("replies-new", { post, comment });
      })
      .catch(err => {
        console.log(err.message);
      });
  });

  // CREATE REPLY
  app.post("/posts/:postId/comments/:commentId/replies", (req, res) => {
    console.log(req.body);
  });
};
```

Next, create your `replies-new` template and have the content sit in the middle 6 columns of the 12 column grid:

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

Our template and form is there, but what happens when we submit it? We already made the create route and left a `console.log(req.body)` --- if you submit that form, you should see the console log the content of the form submission. Do you see it?

The next step is to write our replies#create route logic.

```js
// CREATE REPLY
app.post("/posts/:postId/comments/:commentId/replies", (req, res) => {
  // LOOKUP THE PARENT POST
  Post.findById(req.params.postId)
    .then(post => {
      // FIND THE CHILD COMMENT
      var comment = post.comments.id(req.params.commentId);
      // ADD THE REPLY
      comment.comments.unshift(req.body);
      // SAVE THE CHANGE TO THE PARENT DOCUMENT
      return post.save();
    })
    .then(post => {
      // REDIRECT TO THE PARENT POST#SHOW ROUTE
      res.redirect("/posts/" + post._id);
    })
    .catch(err => {
      console.log(err.message);
    });
});
```

Finally, we need to update our model so that comments have embedded instances of itself.

```js
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const CommentSchema = new Schema({
  content: { type: String, required: true },
  comments: [CommentSchema]
});

module.exports = mongoose.model("Comment", CommentSchema);
```

How strange! And yet, how strange is Reddit itself?

When you submit the reply form, what occurs? Can you confirm (in the database) that a nested embedded comment document is created?

Finally, let's set up our `post-show` template to show these sub comments as well once they are created. If we just try to manually write in our comments and their comments, we won't be able to represent the whole tree. We'll need to use a **Partial Template** to make a recursive representation of all the comments and their comments.

```html
<div class="row">
  {{#each post.comments}}
    {{> partials/comment}}
  {{/each}}
</div>
```

Let's create a new folder in the `views` folder called `partials` and create a file in there called `comment.handlebars`. Inside that template we'll call itself again, so it loops until every comment is displayed.

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
