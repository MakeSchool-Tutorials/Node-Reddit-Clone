---
title: "Comments on Comments"
slug: comments-on-comments
---

Alright next step!

1. Create a post
1. Show all posts
1. Show one post
1. Comment on posts
1. Create subreddits
1. Sign up and Login
1. Associate posts and comments with their author
1. **Make comments on comments**
  1. ... :D
1. Vote a post up
1. Sort posts by # of votes

# Make a Plan

A software developer follows instructions, but a software developer also writes their own instructions! You are writing the instructions

For this step of the tutorial, I'm not going to tell you the step by step, instead you have to make one up yourself. I'm going to write some descriptive text in English about how this feature works in code and you will need to translate that into a few steps.

# Comments on Comments - Reference Documents

One of the best features in Reddit is the ability for users to comment on comments, and for more users to comment on those comments, _ad infinitum_. But how can we store and then recall this endless branching tree of comments?

Currently comments are a reference document on a post. Any comment is a child document of a parent post document. They are connected because a post has a `comments` attribute that is an array. We store the child comments' `_id`'s in this array and then look them up with the `.populate()` mongoose method.

```json
{
  _id: "ji2roi3ji23j2j3",
  user: "fu0fa90faa99a0a",
  body: "Awesome site to share",
  url: "https://www.google.com",
  comments: ["jl3j1kl4j21ljl34", "j234j2l3j42jlk4k3"]
}
```

So what if we tried this same strategy for comments on comments? What would be the pro's and the con's? Would there be problems down stream that would make this pattern unusable? Could we nest comments this way? How would we query a post and all its comments? Would this make it hard to vote on comments? Would it make it hard to count the number of votes an show the most voted on comments on the top? Discuss these questions with a partner who is at this same step.

# Another Strategy: Embedded Documents

Another strategy we could try is using a "[Embedded Document](http://mongoosejs.com/docs/2.7.x/docs/embedded-documents.html)" instead. Embedded documents are actually written into the parent document and are returned every time anyone fetches the parent document.

```json
{
  _id: "ji2roi3ji23j2j3",
  user: "fu0fa90faa99a0a",
  title: "Awesome site to share",
  summary: "But seriously, this is really neat.",
  url: "https://www.google.com",
  comments: [
    {
      _id: "afk3jk4k23j4h232",
      body: "great post"
      comments: [
        {
          _id: "afk3jk4k23j4h232",
          body: "great comment",
          comments: []
        }
      ]
    }
  ]
}
```

The pros of this would be we could nest comments, and we'd always have this giant tree of comments every time we had its parent post record. The con there would be when we asked for just the parent post, we will necessarily have to fetch and then send over the network that entire tree of comments. But so long as we don't have more than a few hundred comments it won't be too slow, and when we do get that kind of traffic (fingers crossed!), we can probably find a solution to speed things up, like excluding the comments field when we query our db for all the posts, and include it when we query for a single post.

# Another Strategy: Reference Comments, Embedded Replies

There is always more than one way to skin a cat.

What if we made comments (the first comments on posts) reference documents, but then we embedded all "replies" or comments on comments inside the top-level comments?

```json
/*POST*/
{
  _id: "ji2roi3ji23j2j3",
  user: "fu0fa90faa99a0a",
  title: "Awesome site to share",
  summary: "But seriously, this is really neat.",
  url: "https://www.google.com",
  comments: ["jl3j1kl4j21ljl34", "j234j2l3j42jlk4k3"]
}

/*COMMENT*/
{
  _id: "ji2roi3ji23j2j3",
  user: "jlk234lj3l432l2k",
  body: "Awesome site to share",
  url: "https://www.google.com",
  replies: [
    {
      _id: "afk3jk4k23j4h232",
      body: "great comment"
      replies: [
        {
          _id: "afk3jk4k23j4h232",
          body: "great sub comment",
          replies: []
        }
      ]
    }
  ]
}
```

This is a more complicated strategy, but are there any advantages? Is it faster or easier to query? Are there routes where we'd want to be able to see just a particular comment and its replies?
