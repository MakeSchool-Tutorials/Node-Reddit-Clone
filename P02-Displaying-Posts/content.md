---
title: "Displaying All Posts"
slug: displaying-posts
---

Alright next step! Now that we can create posts, let's display them.

1. Create a post
1. **Show all posts and show one post**
  1. Make the root route (`/`) go to the posts#index route render a `posts-index` template
  1. Style the template and loop over the `posts` object
  1. Make route to posts#show route (`/posts/:id`)
  1. Style the template and display the `post` object
1. Show one post
1. Comment on posts
1. Create subreddits
1. Sign up and Login
1. Associate posts and comments with their author
1. Make comments on comments
1. Vote a post up

# Connection Script

You'll need to make a connection to Mongoose from `server.js`/`app.js`.

I'm going to encourage the use of promises to handle asynchronous transactions. Mongoose doesn't supply it's own Promise library; instead Mongoose asks you to set a Promise library of your choosing. The tutorial will use the default JavaScript Promise.

Finally, for testing, we can add an error handler for connection errors.

```js
mongoose.Promise = global.Promise;
mongoose.connect(
  "mongodb://localhost/redditclone",
  { useMongoClient: true }
);
mongoose.connection.on("error", console.error.bind(console, "MongoDB connection Error:"));
```

## Bonus

Add the following to display debug info from Mongoose in the console:

```js
mongoose.set('debug', true);
```

# Posts#Index Route

Next, let's have the root route (`/`) render the `posts-index` template.

Can you get the template to display anything like `hello world`, for example? Once you can, we need to then pull the posts out of the database, and send them along with the response.

```js
Post.find({})
  .then(posts => {
    res.render("posts-index.hbs", { posts });
  })
  .catch(err => {
    console.log(err.message);
  });
```

In your template can you output the variable `{{posts}}`?

# Styling and Looping Over Posts

Let's go back to our layout template put the whole `{{{body}}}` object into a div with a container class.

```html
<div class="container">
  {{{body}}}
</div>
```

Now let's put this list of posts into the middle 8 columns of the grid.

```html
<div class="row">
  <div class="col-sm-8 col-sm-offset-2">
    ...
  </div>
</div>
```

Now that we have `{{posts}}`, we can use handlebars' [built in `each` operator](http://handlebarsjs.com/builtin_helpers.html) to loop over the posts, and display each one.

In each post, use bootstrap's `list-group` and `list-group-item` classes. Display the post title in a div with the class `lead`, and add an anchor tag that links to the post's url. Finally, add `target="_blank"` to the anchor tag, so that the url opens in a new tab.

```html
<ul>
  {{#each posts}}
  <li class="list-group-item">
    <div class="lead">{{this.title}}</div>
    <a href="{{this.url}}" target="_blank">{{this.url}}<a>
  </li>
  {{/each}}
</ul>
```

# Viewing One Post

In order to view a single post when a user clicks on it, we'll need to establish a route for individual posts, and render them in their own template.

Let's begin with the **user action** - clicking on a post in the `post-index` template.

```html
<a href="/posts/{{this._id}}" class="lead">{{this.title}}</a>
```

The title is a link to the show page. If we click it, what happens? Error! No route! It's time to fix that.

# Posts#Show Route

We need the path `/posts/:id` to resolve to displaying a `post-show` template. To accomplish this, open `controllers/post.js`, and add a new GET endpoint:

```js
app.get("/posts/:id", function(req, res) {
  // LOOK UP THE POST
  Post.findById(req.params.id)
    .then(post => {
      res.render("post-show.hbs", { post });
    })
    .catch(err => {
      console.log(err.message);
    });
});
```

What happens if we refresh? No template!

# Making the Template

Time to template. As a bare minimum we'll use some bootstrap classes to make things look reasonable as we continue to develop the application.

```html
<div class="row">
  <div class="col-sm-6 col-sm-offset-3">
    <a href="{{post.url}}" class="lead">{{post.title}}</a>
    <p>{{post.summary}}</p>
  </div>
</div>
```

Now can you see your post?
