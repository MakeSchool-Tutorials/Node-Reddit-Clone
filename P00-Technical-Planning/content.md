---
title: "User Narratives, Wire Frames, and Technical Planning"
slug: technical-planning
---

This tutorial is part of a four part series with the [Giphy Search Tutorial](https://www.makeschool.com/online-courses/tutorials/giphy-search-app-with-node-js/your-node-environment), the [Concentration Game](https://www.makeschool.com/online-courses/tutorials/javascript-concentration-game/javascript-game-tutorial-intro), the [Rotten Potatoes Tutorial](https://www.makeschool.com/online-courses/tutorials/rotten-potatoes-movie-reviews-with-express-js/bootstrap-an-express-project). Whereas those tutorials gave you 100% of the code or starter code you needed to build them, this project will be more demanding of you to write your own code, or find the proper code snippets from your other projects and framework documentation.

In this tutorial you will develop a clone of Reddit with posts, subreddits, and authentication. The tools you will use are:

1. Node.js
1. Express.js
1. MongoDB & Mongoose (ODM)
1. JSON Web Tokens (JWT)

This tutorial should take much longer than the previous tutorials to complete and it will require that you read instructions very carefully and follow them step by step. You might encounter errors that you will need to troubleshoot and google to solve. If you would like to provide feedback on this tutorial, please leave GitHub issues on the [GitHub repo](https://github.com/MakeSchool-Tutorials/Node-Reddit-Clone).

# Tutorial structure

This tutorial will walk you through a pattern of "user-centered" or outside-in development with route tests, meaning you will first develop a simple template for a route, then it's actual backend logic. By following this pattern, software is built step-wise and organically without leaps or guesses. This avoids large architectural bugs which are the biggest drain on the speed of a development team.

As you build, keep in mind the rules of a good user interface. Review those here: [goodui.org](http://goodui.org/).

# Getting Started: User Narratives & Wireframes

If you are not familiar with [Reddit](https://www.reddit.com/), navigate there now and explore this internet wonderland. Check out its [history](https://en.wikipedia.org/wiki/Reddit#History) if you like. It was founded by Alexis Ohanian --- now an investor in Make School. Sweet!

While you're there, draw a wireframe of each page on some scratch paper. The basic layout contains two navbars (one for subreddits, and one for sorts/filters) and  a two column layout in a proportion of about 10:2, with no side gutters or padding.

# Technical Planning

Now it's time to get started by making a step-by-step plan. Here are some of the routes and features we could build:

* Create a post
* Show all posts
* Comment on posts
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

In what order should we fulfill these tasks? What do you think? Write it down step by step somewhere. When you are done, hover over the solution to see what order these tasks are implemented from within this tutorial.

We should work from the fundamental to the desirable --- the structure of our program comes first, then any nice-to-haves. Generally, it's best to delay the implementation of authentication as that can cause some initial complexity; we'll do authentication in the middle. Voting could come sooner or later; we put it at the very end. Until that point, we'll have a somewhat-themed Twitter or Facebook wall... until there is voting and sorting on votes, that is!

1. Create a post
1. Show all posts
1. Comment on posts
1. Create subreddits
1. Create a post on a subreddit
1. Show all subreddits
1. Sign up and Login
1. Associate posts, comments with their author
1. Make comments on comments
1. Vote a post up
1. Vote a comment up
1. Sort posts by # of votes

# Bootstrapping Express.js

Ok --- we've got user narratives, wireframes, and a plan for the first few features to implement. Time to get started coding! Remember not to "work ahead", instead always do the absolute minimum work to get what you are working on to function and then move to the next.

1. Create an npm project (hint: `$ npm init`) - remember to make your main file called `server.js`.
1. Bootstrap Express.js. You can use Handlebars or Jade as a templating engine if you like. [Express documentation](https://expressjs.com/en/starter/hello-world.html). Now run your server and get "hello world" to appear.
1. Once you have "hello world" and a template engine installed, move on to the next step.
1. Add Bootstrap to your layout template `<head>` using the CDN. Use the following [starter template](https://getbootstrap.com/docs/4.1/getting-started/introduction/#starter-template) as a guide for your own!

  ```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">

    <title>Reddit.js</title>
  </head>
  <body>
    <h1>Reddit.js</h1>

    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
  </body>
</html>
```

1. Now add the [navbar component](https://getbootstrap.com/docs/4.1/components/navbar/) to your layout template.
1. Onwards!
