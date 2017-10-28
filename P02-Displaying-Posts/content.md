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
1. Sort posts by # of votes

# Connection Script

You'll need to make a connection to Mongoose from server.js/app/js.

I'm going to encourage the use of promises to handle asynchronous transactions. Mongoose doesn't supply it's own Promise library, instead Mongoose asks you to set a Promise library. The tutorial will use the default JavaScript Promise. 

Last, for testing, we can add an error handler for connection errors. 

```js
mongoose.Promise = global.Promise
mongoose.connect('mongodb://localhost/redditclone', { useMongoClient: true })
mongoose.connection.on('error', console.error.bind(console, 'MongoDB connection Error:'))
```

**Bonus**

Add this to get debug info from Mongoose in the console. 

`mongoose.set('debug', true)`

# Posts#Index Route

Let's have the root route (`/`) render the `posts-index` template.

Can you get the template to display anything like `hello world` for example? Once you can we need to now pull the posts out of the database and send them along with the response.

```js
Post.find({}).then((posts) => {
  res.render('posts-index.hbs', { posts })
}).catch((err) => {
  console.log(err.message);
})
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

Now that we have `{{posts}}`, we can use handlebar's [built in `each` operator](http://handlebarsjs.com/builtin_helpers.html) to loop over the posts and display each one.

In each post, let's use bootstrap's `list-group` and `list-group-item` classes. Let's display the post title in a `lead` classed div and add an anchor tag to link to the post's url. Add `target="_blank"` to the anchor tag so that the url opens in a new tab.

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

Now we'd like for each post, when you click on it, to navigate to that particular post's own page.

So let's start with what the user can do - click on a post in the `post-index` template.

```html
<a href="/posts/{{this._id}}" class="lead">{{this.title}}</a>
```

Now the title is a link to the show page. If we click it what do we see? No route! Let's fix that.


# Posts#Show Route

Now we need the path `/posts/:id` to resolve to displaying a `post-show` template. So from inside the posts controller file, we need to have a new GET endpoint.

```js
  app.get('/posts/:id', function (req, res) {
   // LOOK UP THE POST
   Post.findById(req.params.id).then((post) => {
     res.render('post-show.hbs', { post })
   }).catch((err) => {
     console.log(err.message)
   })
 })
```

Now what happens if we refresh? No template!

# Making the Template

Let's get a template in there. As a bare minimum we'll use some bootstrap classes to make things look ok.

```html
<div class="row">
  <div class="col-sm-6 col-sm-offset-3">
    <a href="{{post.url}}" class="lead">{{post.title}}</a>
    <p>{{post.summary}}</p>
  </div>
</div>
```

Now can you see your post? :D
