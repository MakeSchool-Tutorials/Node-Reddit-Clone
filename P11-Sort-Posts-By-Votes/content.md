---
title: "Sort Posts by Votes"
slug: sort-posts-by-votes
---

What's next?

To vote on something, you can usually can vote up and down. "Liking" in Facebook is like voting up, and there is no down. In Reddit however, we can vote things up and down. This is a bit tricky! A few things make it quite tricky.

* When you submit a form with HTML the whole page refreshes, but we don't want that.
* We will only want people to vote once either way
* People generally want to be able to change their votes.

Let's start with how to vote up, and then see how we can vote down too.

1. Create a post
1. Show all posts
1. Show one post
1. Comment on posts
1. Create subreddits
1. Sign up and Login
1. Associate posts and comments with their author
1. Make comments on comments
1. **Vote a post up**
  1. Make vote form
  1. Add jQuery AJAX scripts
  1. Write vote up and vote down routes
  1. Add new attributes to `Post` model
  1. Update DOM with response
  1. Restrict to 1 vote per user
  1. Let people 'undo' their vote
1. Sort posts by # of votes

# Voting Plan

Let's make a plan by doing what we always do and look very carefully at what the user will be able to see and do with our app.

Users should be able to click on up and down arrows to vote a post up or down. The page should not refresh and there should be some indication that you voted up. Once you've voted up or down once, you can't vote up or down again. Ideally, a user could reverse their vote down by voting up to get back to no votes, and then vote up or vise versa, but let's leave this off to the end.

# Submitting a Form Through AJAX

We start by adding a vote up and vote down forms to our post. But this form, remember, needs to be submitted without the default `<form>` behavior of refreshing the page. For that we're going to need to use jQuery and specifically AJAX requests to handle them without refreshing.

**Classes (Not Ids)** - We'll add two forms, tag them with the class `vote-up` and `vote-down`. We'll use these classes as our selectors to fire off the AJAX request to vote up or down.

> [info]
> We use classes here because id's must be unique, but classes can repeat in one html template. In this case, there will be many posts on the posts#index page, so we must use a class instead of an id.

**Data Attribute** - Each of these posts is unique even if there are many on the page. How do we know which one we are voting on? How do we communicate this to the server. In order to solve this problem, we'll be using the `data-id` attribute and render the post's `_id` attribute in each form. Then we can pull that id into the `/posts/:id/vote-up` or `/posts/:id/vote-down` path when we submit the form.

```html
<li class="list-group-item">
  <div class="lead">{{post.title}}</div>
  <a href="{{post.url}}" target="_blank">{{post.url}}<a>
  <div class="text-right">
    <a href="/n/{{post.subreddit}}">{{post.subreddit}}<a>
  </div>

  <form class="vote-up" data-id="{{post._id}}">
    <button type="submit">Vote Up</button>
  </form>
  |
  <form class="vote-down" data-id="{{post._id}}">
    <button type="submit">Vote Down</button>
  </form>
</li>
```

# Adding the AJAX

We're going to add an AJAX request in a new file `public/js/posts.js` (I know another `posts.js` file! Be careful as you are navigating your files.). We already included jQuery into this project when we added Bootstrap, so all our jQuery functions should work splendidly. But we need to add this new script to our `<head>` tag in our `layouts/main.handlebars`

```html
<head>
  ...
  <script rel="script" src="/js/posts.js"></script>

</head>
```

```js
$(document).ready(function() {

  $('.vote-up').submit(function (e) {
    e.preventDefault();

    var postId = $(this).data('id');
    $.ajax({
      type: 'PUT',
      url: 'posts/' + postId + '/vote-up',
      success: function(data) {
        console.log("voted up!");
      },
      error: function(err) {
        console.log(err.messsage);
      }
    });
  });

  $('.vote-down').submit(function (e) {
    e.preventDefault();

    var postId = $(this).data('id');
    $.ajax({
      type: 'PUT',
      url: 'posts/' + postId + '/vote-down',
      success: function(data) {
        console.log("voted down!");
      },
      error: function(err) {
        console.log(err.messsage);
      }
    });
  });

});
```

Now if we click vote up or down, check the Network tab of your Developer Tools and watch each request flying out to your server. Click on one and see what error is coming back. You can even preview the response. Route not found! Great!

# Writing the Vote-Up/Down Routes

Now that we are submitting to the routes, we have to write them. Let's put them in the top of the `controllers/posts.js` file.

Reminder: we're using PUT because we are editing an existing resource.

We want to track who voted on what, and we want to know what the total score of votes is. So we can pretend the `Post` model has three new attributes: `upVotes`, `downVotes`, and `voteScore`.

```js
app.put('posts/:id/vote-up', function (req, res) {
  Post.findById(req.params.id).exec(function (err, post) {
    post.upVotes.push(req.user._id)
    post.voteScore = post.voteTotal + 1
    post.save();

    res.status(200);
  })
})

app.put('posts/:id/vote-down', function (req, res) {
  Post.findById(req.params.id).exec(function (err, post) {
    post.downVotes.push(req.user._id)
    post.voteScore = post.voteTotal - 1
    post.save();

    res.status(200);
  })
})
```
