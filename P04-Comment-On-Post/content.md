---
title: "Displaying All Posts"
slug: displaying-posts
---

Alright next step! Now that we can create posts, let's display them.

1. Create a post
1. Show all posts
1. Show one post
1. **Comment on posts**
  1.
1. Create subreddits
1. Create a post on a subreddit
1. Show all subreddits
1. Sign up and Login
1. Associate posts, comments with their author
1. Delete posts
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

Create a new file `comments.js` in your `controllers` folder and follow the pattern you used for the `posts.js` file to

1. Require the comment model
1. Export the comments controller into the `server.js`
