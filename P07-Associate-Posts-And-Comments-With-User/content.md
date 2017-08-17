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
1. Vote a comment up
1. Sort posts by # of votes

# Add the `author` Attribute to Comment and Post Models

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
