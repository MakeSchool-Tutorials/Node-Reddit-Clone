---
title: "Associate Posts and Comments with User"
slug: associate-posts-and-comments-with-user
---

Alright next step! Its time to allow people to take responsibility for the silly things they are posting on our Reddit clone. We are going to create user accounts so folks can securely signup and login using a username and password.

1. Create a post
1. Show all posts
1. Show one post
1. Comment on posts
1. Create subreddits
1. Sign up and Login
1. **Association posts and comments with their author**
    1. Check authentication and make `req.user` and `currentUser` objects
    1. Add `author` attribute to comments and posts
    1. Save the user as the author of posts
    1. Display the author's username on posts and comments
1. Make comments on comments
1. Vote a post up or down

# Checking Login Status

So now that we are setting the cookie to be logged in, and can see this cookie on the server or client side, how do we customize our views and the **authorization** of a user? There are a few ways to do it. We could use client JavaScript to track the presence of the `nToken` cookie and update the DOM based on that. But let's try to find a server-side solution first because our app is very server-side structured now.

We can always check if `req.cookies.nToken` is present, but shouldn't we also check if it is valid? And don't we really want the user `_id`of the user this token represents? This is a lot of code to include in every route! In order to refactor this code, we can make our own custom middleware and put it in `server.js` so it is used for every route.

```js
var checkAuth = (req, res, next) => {
  console.log("Checking authentication");
  if (typeof req.cookies.nToken === "undefined" || req.cookies.nToken === null) {
    req.user = null;
  } else {
    var token = req.cookies.nToken;
    var decodedToken = jwt.decode(token, { complete: true }) || {};
    req.user = decodedToken.payload;
  }

  next();
};
app.use(checkAuth);
```

Go to any route and see if the `Checking authentication` is logged in the terminal.

Note: this is your own custom middleware like `body-parser` and `cookie-parser`. Middleware is run in the order it is added via `app.use()`. You must add middleware after initializing Express, and the order will matter.

# Updating Templates

Once our `checkAuth` middleware is running on every route, let's use the new `req.user` object we created to create a `currentUser` object. We'll use this `currentUser` object to hid the login and sign up links if a user is logged in.

```html
<ul class="nav navbar-nav navbar-right">
  {{#if currentUser}}
    <li><a href="/logout">Logout</a></li>
  {{else}}
    <li><a href="/login">Login</a></li>
    <li><a href="/sign-up">Sign Up</a></li>
  {{/if}}
</ul>
```

Now in any route we can set `currentUser` equal to `req.user` which will either be `{ _id: <<ID>> }` or `null`.

```js
app.get("/", (req, res) => {
  var currentUser = req.user;

  Post.find({})
    .then(posts => {
      res.render("posts-index.hbs", { posts, currentUser });
    })
    .catch(err => {
      console.log(err.message);
    });
});
```

Do the login and sign up links appear and disappear depending upon whether a user is logged in?

Now, hide the "New Post" button for those who are logged in.

```html
{{#if currentUser}}
  <a href="/posts/new" class="btn btn-primary navbar-btn">New Post</a>
{{/if}}
```

Remember, you'll have to add `currentUser` to all of the routes that call `res.render()` so the main templates work. This may seem like some duplication of code, and it is. Can you brainstorm a 100% DRY solution with a friend, without code duplication?

# Requiring User to Be Logged In to Post

Right now, if you aren't logged in, you could still just navigate to `/posts/new` and create a post. Let's be a bit more secure and prevent someone from creating a post unless they are logged in.

```js
// CREATE
app.post("/posts", (req, res) => {
  if (req.user) {
    var post = new Post(req.body);

    post.save(function(err, post) {
      return res.redirect(`/`);
    });
  } else {
    return res.status(401); // UNAUTHORIZED
  }
});
```

We could do this more elegantly and more DRY if we made another example of [custom middleware called](https://expressjs.com/en/guide/writing-middleware.html) `CheckAuth` and used it on those routes that we require to be logged in.

Can you rewrite the above code into its own middleware called `CheckAuth`?

# Associating the `author` of Comments and Posts

So now we want people to take responsibility for their silly posts on Reddit.js.

We need to make each post and each comment point back to it's author, as well as ensure the posts and comments are persisted with associations to the user that creates them. We'll do this by saving the author's user id in each child post or comment, and by tracking the post and comment id's for each user. Then we can use the `.populate()` method to pull in the details whenever we need.

To accomplish this, there are no changes required to the views we've already created. We can go straight to updating model and controller appropriately.

First, let's add an `author` attribute to both the `models/comment.js` and the `models/post.js` files. It's type will be a single `ObjectId`. We'll make it required because only logged in people can create posts.

```js
...
 author : { type: Schema.Types.ObjectId, ref: "User", required: true }
...
```

Additionally, add the `posts` attribute to the `User` model. It will be an array of `ObjectId`s.

```js
  ...
  posts : [{ type: Schema.Types.ObjectId, ref: "Post" }]
  ...
```

Now we can update the posts controller to save the current user as the author when we create a post,
and we can look up the current user and add the new post to their `posts`.

```js
var post = new Post(req.body);
post.author = req.user._id;

post
  .save()
  .then(post => {
    return User.findById(req.user._id);
  })
  .then(user => {
    user.posts.unshift(post);
    user.save();
    // REDIRECT TO THE NEW POST
    res.redirect("/posts/" + post._id);
  })
  .catch(err => {
    console.log(err.message);
  });
```

Test that both the `author` and the `posts` are being saved by looking in your database or logging to the console.

Now, can you do this same pattern for the comments controller when someone creates a comment?

# Displaying the Author

Next, populate the author in posts and display their username on every post wherever it appears.

# Now For Comments

Using the previous instructions for associating users and posts, can you make it so users and comments are equally associated?

# Stretch Challenges

1. Can you make an author's username a link that displays that users's profile at `/users/:username`?
1. Can you do the same for comments?
1. Can you make a `/profile` route that loads the current user and displays their posts and comments?
