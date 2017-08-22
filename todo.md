Error handling
```js
app.get('/', (request, response) => {  
  throw new Error('oops')
})

app.use((err, request, response, next) => {  
  // log the error, for now just console.log
  console.log(err)
  response.status(500).send('Something broke!')
})
```

Testing
Validation

require the comment model into the proper controllers

How do you create a new reply comment on an embedded comment?


Challenges:

1. Show all subreddits
1. Delete posts
1. What other ways could you sort your posts
  - recently posted DONE
  - most votes DONE
  - most active (most votes & comments)
  - most commented on
  - recently active (votes or comments)
  - ???
1. Authorization. Edit/Update/Delete only your own comments and posts
1. Protect paths that you must be logged in to use
1. Add user id's to comments
1. password reset


# Authorization

Now that we know whose posts and comments are whose, we can do some **Authorization**.

> [info]
> **Authorization** means we are allowing some users to do certain things and preventing others. Some examples are admin users might be able to do all kinds of things, and normal users cannot. A common example of authorization is only allowing people to edit or delete instances they created.

We're going to prevent someone from commenting on their own post.

We'll do it right in the view. If we wanted to be doubly safe that no one could do this action we could also block it in
