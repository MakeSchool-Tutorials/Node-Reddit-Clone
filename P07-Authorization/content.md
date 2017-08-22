---
title: "Authorization"
slug: authorization
---

Now that we can sign up, login, and logout, its time to add some authorization.

1. Create a post
1. Show all posts
1. Show one post
1. Comment on posts
1. Create subreddits
1. Sign up and Login
1. **Authorization** - added one!
1. Associate posts and comments with their author
1. Make comments on comments
1. Vote a post up
1. Sort posts by # of votes

# Authorization: Templates



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
