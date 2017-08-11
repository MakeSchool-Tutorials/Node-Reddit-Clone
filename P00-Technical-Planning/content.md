---
title: "User Narratives, Wire Frames, and Technical Planning"
slug: technical-planning
---

This tutorial is part of a three part series with the Giphy Search Tutorial and the Rotten Potatoes Tutorial. Whereas those tutorials gave you 100% of the code or starter code you needed to build them, this project will be more demanding of you to write your own code, or find the proper code snippets from your other projects and framework documentation.

In this tutorial you will develop a clone of Reddit with posts, subreddits, authentication, and even a search bar. The tools you will use are:

1. Node.js
1. Express.js
1. MongoDB & Mongoose (ODM)
1. JSON Web Tokens (JWT)

This tutorial should take much longer than those to complete and it will require that you read instructions very carefully and follow them step by step. You might encounter errors that you will need to troubleshoot and google to solve. If you would like to provide feedback on this tutorial, please leave github issues on the [github repo](https://github.com/MakeSchool-Tutorials/Node-Reddit-Clone).

# Tutorial structure

This tutorial will walk you through a pattern of "user-centered" or outside-in development with route tests, meaning you will first develop a simple template for a route, then its actual backend logic. By following this pattern software is built step-wise and organically without leaps or guesses. This avoids large architectural bugs which are the biggest drain on the speed of a dev team.

As you design your interfaces, use inspirations (like Reddit.com!), and keep in mind the rules of a good user interface - you can review those here: [goodui.org](http://goodui.org/).

# Getting Started: User Narratives & Wireframes

If you are not familiar with [Reddit](https://www.reddit.com/), navigate there now and explore this internet wonderland. Check out its [history](https://en.wikipedia.org/wiki/Reddit#History) if you like. It was founded by Alexis Ohanian who is now an investor in Make School. Kinda neat!

While you're there, draw a wireframe of each page on some scratch paper. The basic layout is two navbars (one for subreddits and one for sorts/filters) and then a two column layout in a proportion of about 10:2, with no side gutters or padding.

# Technical Planning

Now its time to get started by making a step-by-step plan.

Here are some of the routes and features we could build

* Comment on posts
* Create a post
* Show all posts
* Make comments on comments
* Sign up
* Login
* Associate posts, comments, and votes with their author
* Search
* Create subreddit
* Create a post on a subreddit
* Show all subreddits
* Vote up a post
* Vote up a comment
* Sort posts by # of votes

In what order should we fulfill these tasks? Write the order down somewhere that you would do these tasks. When you are done, hover over the solution to see what order these tasks in this tutorial.

> [solution]
> We should work from the most fundamental out to things that are more wants and not needs. Here's what we'll do this this tutorial. It is generally best to not start with authentication because that can cause some initial complexity, so we'll do authentication in the middle. Voting could come sooner or later, I put it at the very end so we'll have a kind of themed twitter or facebook wall until there is voting and sorting on votes.
> 1. Create a post
> 1. Show all posts
> 1. Comment on posts
> 1. Create subreddits
> 1. Create a post on a subreddit
> 1. Show all subreddits
> 1. Sign up and Login
> 1. Associate posts, comments with their author
> 1. Make comments on comments
> 1. Vote a post up
> 1. Vote a comment up
> 1. Sort posts by # of votes

# Bootstrapping Express.js

Ok, we've got user narratives, wireframes, and a plan for the first few features we'll do. Time to get started coding! Remember not to "work ahead", instead always do the absolute minimum work to get what you are working on to function and then move to the next.

1. Create an npm project (hint: `$ npm init`) - remember to make your main file called `server.js`.
1. Bootstrap Express.js. You can use Handlebars or Jade as a templating engine if you like. [Express documentation](https://expressjs.com/en/starter/hello-world.html). Now run your server and get "hello world" to appear.
1. Once you have "hello world" and a template engine installed, move on to the next step.
1. Now add bootstrap to your layout template `<head>` using the CDN.
  ```html
  <!-- Latest compiled and minified CSS -->
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">

  <!-- Latest compiled and minified JavaScript -->
  <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"></script>
  ```
1. Now add the [navbar component](http://getbootstrap.com/components/#navbar) to your layout template.
1. Onwards!
