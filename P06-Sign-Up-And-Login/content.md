---
title: "Sign Up, Logout, and Login"
slug: sign-up-and-login
---

Alright next step! Its time to allow people to take responsibility for the silly things they are posting on our Reddit clone. We are going to create user accounts so folks can securely signup and login using a username and password.

1. Create a post
1. Show all posts
1. Show one post
1. Comment on posts
1. Create subreddits
1. **Sign up and Login**
  1. Make `/sign-up` route, template and form
  1. Make POST `/sign-up` route and logic
  1. Make `User` model
  1. Encrypt users' passwords
  1. Create JWT and add Cookies
  1. Demonstrate that user is logged in and password is encrypted
  1. Make `/login` route, template and form
  1. Make POST `/login` route and logic
1. Associate posts and comments with their author
1. Make comments on comments
1. Vote a post up
1. Vote a comment up
1. Sort posts by # of votes

# Make `/sign-up` Route

Always think first about what the user's experience should be, and then develop for that. We'll follow the convention of a "sign up / login" links living in the navbar of our project.

```html
```

Now that we have the links, let's make the `/sign-up` route work. We can make a new controller file called `auth.js` to put all our authentication routes.

```js

```

Now we can create our sign up form with `username` and `password` fields.

# Make the POST Route to `/sign-up`

That form won't work though because there is no POST `/sign-up` route yet. Let's make it

```js

```

Oh no! We don't have a user model yet. Let's make a new file called `user.js` on your `models` folder.

```js

```

# Encrypting the Password

The special issue we have with users is we can't know the users' password, we have to encrypt it. To do that we'll use a project called `bcrypt` that gives us some easy methods to encrypt passwords and save a secure "password digest" instead of the actual password.

All this bcrypt logic will live in the user model.

```js

```

Now let's sign up and check if our password was encrypted. We'll test if the encryption actually worked when we create the login form

# Create JWT Tokens and Cookies

Besides just creating a user document when the user signs up, we also want to log that user in to our application. What does it mean to be logged in? To be authenticated?

In our case, we're going to use a JWT token - a JSON Web Token. This token is another piece of neat encryption that uniquely identifies a specific user. We'll use a few separate libraries to generate this token, and then we'll use `cookieParser` to set this token as a cookie.

First we will create the JWT token.

https://stackoverflow.com/questions/3393854/get-and-set-a-single-cookie-with-node-js-http-server

```js

```

Next we'll set the cookie.

```js

```

Now lets see if the cookie is set by examining the cookies in another request, say just to the root route `/`.

```js

```

# Let's Log out

Now that we have signed up, let's log out.

# Let's Login

Now that we've signed up, logged out, now let's login. We can use the same pattern as we did with the `/sign-up` routes, with one GET and one POST both with the path `/login`.

First let's build the GET route and template

```js

```

```html

```

Now let's make the logic for the POST route to `/login` work.

```js

```

If we did everything right, now we should be able to sign up, log out, and login securely. Nice! In the next chapter we'll associate our user record with our posts and comments.

WOOOHOOO!
