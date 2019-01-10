---
title: "Testing Authentication"
slug: testing-authentication
---

So we can do authentication, but what if the next time we turn around, this feature breaks? How would we catch it? The answer is more automated tests.

So it is really fun to test things right!?

Maybe not so fun, but writing tests is a critical skill for a young engineer. At almost all companies all code must have tests that go along with it. And at many companies, young engineers are tasked with writing tests and crushing bugs for their first few weeks, so knowing how to test well will make you a very hirable and quick-to-ramp employee.

# Testing Authentication

Testing RESTful routes for a regular resource is pretty straightforward. There might be some quirks, but pretty much you just `GET`, `POST`, `PUT`, and `DELETE` data and verify that the responses are successful.

With authentication, however, testing gets a little more complicated. You have to check that `JWTs` are creating, that passwords are correct, and then you have to be prepared to test authorization.

The biggest challenge is tracking the `cookie` set by the server when a user logs in. In order to handle this `cookie` we can use `chai`'s [request agent](https://github.com/chaijs/chai-http#retaining-cookies-with-each-request) functionality to track the cookie in our tests.

Before we can run **any** of our `auth` test, we're going to need to add a line to `server.js` that exports our `app` variable that `mocha` needs in order to successfully run our `auth` tests.

> [action]
> Add this line to the bottom of `server.js`:
>
```js
module.exports = app;
```

Now our `auth` tests can be run!

> [action]
> Create the file `test/auth.js`. Within, we'll require the libraries we're going to need.
>
```js
var chai = require("chai");
var chaiHttp = require("chai-http");
var server = require("../server");
var should = chai.should();
chai.use(chaiHttp);
>
var agent = chai.request.agent(server);
>
var User = require("../models/user");
>
describe("User", function() {
  // TESTS WILL GO HERE.
});
```

We can now write our first test that verifies that you cannot login if you haven't signed up yet.

> [action]
> Add this test within your `User` block:
>
```js
it("should not be able to login if they have not registered", done => {
  agent.post("/login", { email: "wrong@wrong.com", password: "nope" }).end(function(err, res) {
    res.status.should.be.equal(401);
    done();
  });
});
```

Can you make it not pass, and then pass?

# Testing Sign Up, Logout, and Login

What should we test next? Sign up! Read the following code very carefully, then add it to your project. Can you make the test red (not pass) and then green (pass)?

> [action]
> Add this `signup` test to `test/auth.js`:
>
```js
// signup
it("should be able to signup", done => {
  User.findOneAndRemove({ username: "testone" }, function() {
    agent
      .post("/sign-up")
      .send({ username: "testone", password: "password" })
      .end(function(err, res) {
        console.log(res.body);
        res.should.have.status(200);
        agent.should.have.cookie("nToken");
        done();
      });
  });
});
```

Next, write a test that verifies that your logout implementation works properly:

> [action]
> Add this `logout` test to `test/auth.js`:
>
```js
// logout
it("should be able to logout", done => {
  agent.get("/logout").end(function(err, res) {
    res.should.have.status(200);
    agent.should.not.have.cookie("nToken");
    done();
  });
});
```

Finally, we write a test to verify that the login functionality works as expected.

> [action]
> Add this `login` test to `test/auth.js`:
>
```js
// login
it("should be able to login", done => {
  agent
    .post("/login")
    .send({ username: "testone", password: "password" })
    .end(function(err, res) {
      res.should.have.status(200);
      agent.should.have.cookie("nToken");
      done();
    });
});
```

# Authorization in Other Tests

Now that we have authentication we're going to want to make it so some routes only work if a user is authenticated. Maybe you can only create posts if you are logged in. This means we'll have to update our tests to login before we try to create a post.

In order for the test agent to be logged in, you have to use a `before` action that mocha gives you. The `before` action runs before all the tests run.

```js
// test/posts.js

before(done => {
  agent
    .post("/login")
    .send({ username: "testone", password: "password" })
    .end(function(err, res) {
      done();
    });
});
```

Now the agent has the JWT cookie when it makes other requests like to view the `/posts/new` route or create a post.

This is good we have this now, because when we made it impossible to create posts when a user was not logged in broke the `/posts/create` route test!

# Now Commit

```bash
git add .
git commit -m 'Implemented authentication tests'
git push
```

# Stretch Challenges

1. Can you use this code or something like it to fix the `/posts/create` test?
1. Can you write another test to test that it is impossible to create a post if a user is not logged in?
