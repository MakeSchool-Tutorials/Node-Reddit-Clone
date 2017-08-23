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
1. **Associate posts and comments with their author**
  1. Add `author` attribute to comments and posts
  1. Save the user as the author of posts
  1. Remove the ability to comment on one's own posts
  1. Display the author's username on posts and comments
1. Make comments on comments
1. Vote a post up
1. Sort posts by # of votes


# Checking Login Status

So now that we are setting the cookie to be logged in, and can see this cookie on the server or client side, how do we customize our views and the **authorization** of a user? There are a few ways to do it. We could use client JavaScript to track the presence of the `nToken` cookie and update the DOM based on that. But let's try to find a server-side solution first because our app is very server-side structured now.

We can always check if `req.cookies.nToken` is present, but shouldn't we also check if it is valid? And don't we really want the user `_id`of the user this token represents? This is a lot of code to include in every route! In order to refactor this code, we can make our own custom middleware and put it in `server.js` so it is used for every route.

```js
var checkAuth = function (req, res, next) {
  console.log("Checking authentication");

  if (typeof req.cookies.nToken === 'undefined' || req.cookies.nToken === null) {
    req.user = null;
  } else {
    var token = req.cookies.nToken;
    var decodedToken = jsonwebtoken.decode(token, { complete: true }) || {};
    req.user = decodedToken.payload;
  }

  next()
}

app.use(checkAuth)
```

Go to any route and see if the `Checking authentication` is logged in the terminal.


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

Now in any route we can set `currentUser` equal to req.user which will either be `{ _id: <<ID>> }` or `null`.

```js
app.get('/', function (req, res) {
  var currentUser = req.user;

  Post.find().exec(function (err, posts) {
    res.render('posts-index', { posts: posts, currentUser: currentUser });
  });
})
```

Do the login and sign up links appear and disappear if you are logged in or not?

Add `currentUser` to all of the routes that call `res.render()` so the templates can use it.

# Associating the `author` of Comments and Posts

Let's add an `author` attribute to both the `comment.js` and the `post.js` files and let's make it required:

```js
...
, author         : { type: Schema.Types.ObjectId, ref: 'User', required: true }
...
```

# Save the Current User as the Author of Posts and Comments

Now we can update the posts controller to save the current user as the author when we create a post.

```js
User.findById(req.user._id).exec(function (err, user) {
  var post = new Post(req.params.tourId);

  post.save(function (err, post) {
    // REDIRECT TO THE NEW POST
    res.redirect('/posts/'+ post._id)
  });
});

```

Test that this is working in your project.

Now can you do this same pattern for the comments controller for when someone creates a comment?

# Displaying the Author

Now populate the author in posts and display their username on every post wherever it appears.

# Extra Challenges:

1. Can you make an author's username a link that displays that users's profile at `/users/:username`?
1. Can you do the same for comments?
