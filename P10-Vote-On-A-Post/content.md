---
title: "Vote On a Post"
slug: vote-on-a-post
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
  1. Add `votes` attribute
  1. Restrict to 1 vote per user
1. Sort posts by # of votes

# Voting Plan

Let's make a plan by doing what we always do and look very carefully at what the user will be able to see and do with our app.

Users should be able to click on up and down arrows to vote a post up or down. The page should not refresh and there should be some indication that you voted up. Once you've voted up or down once, you can't vote up or down again. Ideally, a user could reverse their vote down by voting up to get back to no votes, and then vote up or vise versa, but let's leave this off to the end.

# Submitting a Form Through AJAX

We start by adding a vote up form to our post. But this form, remember, needs to be submitted via AJAX without 
refreshing the page. 

```html
<li class="list-group-item">
  <div class="lead">{{post.title}}</div>
  <a href="{{post.url}}" target="_blank">{{post.url}}<a>
  <div class="text-right">
    <a href="/n/{{post.subreddit}}">{{post.subreddit}}<a>
  </div>
  <form id="vote-up">

  </form>
</li>
```
