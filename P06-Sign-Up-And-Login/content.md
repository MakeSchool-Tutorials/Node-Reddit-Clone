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
1. Sort posts by # of votes

# Make `/sign-up` Route

Always think first about what the user's experience should be, and then develop for that. We'll follow the convention of a "sign up / login" links living in the navbar of our project.

```html
<nav class="navbar navbar-default">
  <div class="container">
    ...
    <ul class="nav navbar-nav navbar-right">
      <li><a href="/login">Login</a></li>
      <li><a href="/sign-up">Sign Up</a></li>
    </ul>
  </div>
</div>
```

Now that we have the links, let's make the `/sign-up` route work. We can make a new controller file called `auth.js` to put all our authentication routes.

```js
// SIGN UP FORM
app.get('/sign-up', function(req, res, next) {
  res.render('sign-up');
});
```

Now we can create our sign up form with `username` and `password` fields.

# Make the POST Route to `/sign-up`

Use the template code you used to create a post but alter it to fit a username/password form. Don't forget:

1. Set the password input field type to `password`
1. Set the action of the form to `/sign-up`
1. Put the form in the middle third of the grid.

If you submit the form, you'll see that there is no POST `/sign-up` route yet. Let's make it.

```js
// SIGN UP POST
app.post('/sign-up', function(req, res, next) {
  // Create User and JWT
  var user = new User(req.body);

  user.save(function (err) {
    if (err) { return res.status(400).send({ err: err }) }

    res.redirect('/');
  })
});
```

Oh no! We don't have a user model yet. Let's make a new file called `user.js` on your `models` folder.

```js
var mongoose = require('mongoose'),
    Schema = mongoose.Schema;

var UserSchema = new Schema({
    createdAt       : { type: Date }
  , updatedAt       : { type: Date }

  , password        : { type: String, select: false }
  , username        : { type: String, required: true }
});

UserSchema.pre('save', function(next){
  // SET createdAt AND updatedAt
  var now = new Date();
  this.updatedAt = now;
  if ( !this.createdAt ) {
    this.createdAt = now;
  }
});

module.exports = mongoose.model('User', UserSchema);
```

Now we have a user model, but if we save our password it will be saved in our database as plaintext and not be secure.

# Encrypting the Password

The special issue we have with users is we can't know the users' password, we have to encrypt it. To do that we'll use a project called `bcrypt` that gives us some easy methods to encrypt passwords and save a secure "password digest" instead of the actual password.

All this bcrypt logic will live in the user model. What we'll do is:

1. Install `bcrypt` to our project and require it in our model
1. Add a method to our user model that detects if the `password` attribute is being modified, and if it is salt and has the password to produce a password digest that we'll save into the database.
1. Lastly, make a model method called `comparePassword()` that takes in an attempted password and returns true or false if the attempt matches what is in the database.

Read this implementation closely and add implement the same into your `User` model.

```js
var mongoose = require('mongoose'),
    bcrypt = require('bcrypt'),
    Schema = mongoose.Schema;

var UserSchema = new Schema({
    createdAt       : { type: Date }
  , updatedAt       : { type: Date }

  , password        : { type: String, select: false }
  , username        : { type: String, required: true }
});

UserSchema.pre('save', function(next){
  // SET createdAt AND updatedAt
  var now = new Date();
  this.updatedAt = now;
  if ( !this.createdAt ) {
    this.createdAt = now;
  }

  // ENCRYPT PASSWORD
  var user = this;
  if (!user.isModified('password')) {
    return next();
  }
  bcrypt.genSalt(10, function(err, salt) {
    bcrypt.hash(user.password, salt, function(err, hash) {

      user.password = hash;
      next();
    });
  });
});


UserSchema.methods.comparePassword = function(password, done) {
  bcrypt.compare(password, this.password, function(err, isMatch) {
    done(err, isMatch);
  });
};

module.exports = mongoose.model('User', UserSchema);
```

Now let's sign up and check if our password was encrypted. We'll be able to see if the password is salted and hashed, and later we'll be able to test if the encryption actually worked when we create the login form.

# Create JWT Tokens and Cookies

Besides just creating a user document when the user signs up, we also want to log that user in to our application. What does it mean to be logged in? To be authenticated?

In our case, being logged in will mean that there is an authentic JWT token - a JSON Web Token - set as a cookie. This token is another piece of neat encryption that uniquely identifies a specific user.We'll use a one library called `jsonwebtoken` to generate this token, and then we'll use `cookie-parser` to set this token as a cookie.

```bash
$ npm install cookie-parser jsonwebtoken -s
```



First we generate JSON Web Tokens (JWTs) so we need to require the npm module `jsonwebtoken` we just installed into our `auth.js` controller and then use it to generate a JWT after the new user document is saved.

```js
var jwt = require('jsonwebtoken');

...

user.save(function (err) {
  if (err) { return res.status(400).send({ err: err }) }

  var token = jwt.sign({ _id: user._id }, process.env.SECRET, { expiresIn: "60 days" });

  res.redirect('/');
})

```

Next we need to set the JWT as a cookie so that it will be included in all future requests from the current user's client. First we include cookieParser in the project in the `server.js` file so cookie methods can be used anywhere in the app.

```js
var express = require('express')
var cookieParser = require('cookie-parser')

var app = express()
app.use(cookieParser())
```

Next we'll set the cookie. (We'll want our JWT cookie variable's name to bit unique so it doesn't get confusing with other sites, so we'll use the variable `nToken`.)

```js
user.save(function (err) {
  if (err) { return res.status(400).send({ err: err }) }

  var token = jwt.sign({ _id: user._id }, process.env.SECRET, { expiresIn: "60 days" });
  res.cookie('nToken', token, { maxAge: 900000, httpOnly: true });

  res.redirect('/');
})
```

Now lets see if the cookie is set by examining the cookies in another request, say just to the root route `/`.

```js
  console.log(req.cookies);
```

You can also see this cookie in the client under Developer Tools > Application tab > Cookies, or by typing in `document.cookies` in the client console.

# Authorization - Checking Login

So now that we are setting the cookie to be logged in, and can see this cookie on the server or client side, how do we customize our views and the **authorization** of a user? There are a few ways to do it. We could use client JavaScript to track the presence of the `nToken` cookie and update the DOM based on that. But let's try to find a server-side solution first because our app is very server-side structured now.

We can always check if `req.cookies.nToken` is present, but shouldn't we also check if it is valid? And don't we really want the user `_id`of the user this token represents? This is a lot of code to include in every route! In order to refactor this code, we can make our own custom middleware and put it in `server.js` so it is used for every route.

```js
var checkAuth = function (req, res, next) {
  console.log("Checking authentication");

  if (typeof req.cookies.nToken === 'undefined' || req.cookies.nToken === null) {
    req.user = null;
  } else {
    var token = req.cookies.nToken;
    var decodedToken = jsonwebtoken.decode(token, { complete: true }) || {};
    req.user = decodedToken.payload;
  }

  next()
}

app.use(checkAuth)
```

Go to any route and see if the `Checking authentication` is logged in the terminal. Once it is, let's use the new `req.user` object to create a `currentUser` object that we use to hid the login and sign up links if a user is logged in.

```html
<ul class="nav navbar-nav navbar-right">
  {{#unless currentUser}}
    <li><a href="/login">Login</a></li>
    <li><a href="/sign-up">Sign Up</a></li>
  {{/unless}}
</ul>
```


```js
app.get('/', function (req, res) {
  var currentUser = req.user;

  Post.find().exec(function (err, posts) {
    res.render('posts-index', { posts: posts, currentUser: currentUser });
  });
})
```

Add `currentUser` to all of the routes that call `res.render()` so the templates can use it.

# Let's Logout

Now that we have signed up, let's log out. Since "being logged in" just means that the cookie is set, we can create a new `/logout` route that just removes this cookie.

> [info]
> It might be more accurate to use the DELETE method, because we are sort of deleting something, but we will just use the get method to simplify our requests at this point.

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

```js
// LOGOUT
app.get('/logout', function(req, res, next) {
  res.clearCookie('nToken');

  res.redirect('/');
});
```

Are the Login and Sign Up links back?

# Let's Login

Now that we've signed up, logged out, now let's login. We can use the same pattern as we did with the `/sign-up` routes, with one GET and one POST both with the path `/login`.

First let's build the GET route and template

```js
// LOGIN FORM
app.get('/login', function(req, res, next) {
  res.render('login');
});
```

Use the `sign-up` template code to make a login form.

Now let's make the logic for the POST route to `/login` work.

```js
// LOGIN
app.post('/login', function(req, res, next) {
  User.findOne({ email: req.body.email }, "+password", function (err, user) {
    if (!user) { return res.status(401).send({ message: 'Wrong email or password' }) };
    user.comparePassword(req.body.password, function (err, isMatch) {
      if (!isMatch) {
        return res.status(401).send({ message: 'Wrong email or password' });
      }

      var token = jwt.sign({ _id: user._id }, process.env.SECRET, { expiresIn: "60 days" });
      res.cookie('nToken', token, { maxAge: 900000, httpOnly: true });

      res.redirect('/');
    });
  })
});

```

Phew!

If we did everything right, now we should be able to sign up, log out, and login securely. We even got a little bit of authorization working showing and hiding the login, sign up, logout links. Nice! In the next chapter we'll associate our user record with our posts and comments.

WOOOHOOO!

# Stretch: More Authentication Patterns

There is a lot more to make a full fledged authentication system. See which ones you want to try:

1. Add a Remember Me checkbox. What is the difference from when it is checked or not?
1. Require a password confirmation field.
1. Plan out how you would do a "forget password" process.
