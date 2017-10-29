---
title: "Testing Posts"
slug: testing-posts
---

So now we can create and show posts, but we are never done with code until we've written automated tests for it. So we'll have to setup our test environment and make a few controller- or route-level tests.

# Installing Mocha and Chai & Hello World Test

Mocha.js is a test framework for Node.js, and Chai.js is an assertion library that does the work of determining whether a test passes or fails. `chai-http` makes it easy for us to make test http requests to our server. Let's add them to our project and run our first test.

We're going to add these to our `devDependencies` in our `package.json` file because we don't need the testing libraries in production.

```bash
$ npm install mocha chai chai-http --save-dev
```

Now create a folder called `test` in the root of your project.

Add a file called `index.js` and let's require our testing libraries and then create our first hello world style test.

```js
const chai = require('chai')
const chaiHttp = require('chai-http')
const should = chai.should()

chai.use(chaiHttp)

describe('site', () => {                  // Describe what you are testing
  it('Should have home page', (done) => { // Describe what should happen
    // In this case we test that the home page loads
    chai.request('localhost:3000')
      .get('/')
      .end((err, res) => {
        if (err) {
          done(err)
        }
        res.status.should.be.equal(200)
        done()                           // Call done if the test completed successfully.
      })
  })
})
```

This test tests that the reponses status should be equal to 200 - which if you recall your HTTP status codes, means the response is successful.

Now let's run the test. First update your `package.js` file to have a test command:

```json
"scripts": {
  "test": "mocha"
},
```

In order for this test to run the server will have to be running on localhost 3000. To do this you will need to have two terminal windows open. Run your server in one then run the script below in the other. 

Now we can run our tests with:
```bash
$ npm run test
```

What was the result? Can you make the test fail?

# Testing Posts#Create

Next let's make a test for the posts#create route we made. We can make a new file in `test` called `posts.js`.

```js
...
const Post = require('../models/post')
...

describe('Posts', () => {
  it('should create with valid attributes at POST /posts', (done) => {
    // test code
  })
})
```

The order of pseudocode we want to see is as follows

```js
// How many posts are there now?
// Make a request to create another
// Check that the database has one more post in it
// Check that the response is a successful
```

So if we write that in:

```js
// Import your Post model
// How many tours are there now?
Tour.find(function(err, tours) {
  var tourCount = tours.count;

  var tour = { title: "post title", url: "https://www.google.com", summary: "post summary" }
  chai.request('localhost:3000')
    .post('/tours', tour)
    .end(function (err, res){

      // Check that the database has one more tour in it
      Tour.find(function(err, tours) {
        tourCount.should.be.equal(tours + 1);

        // Check that the response is a successful
        res.should.have.status(200);
      done();
    });
  });
});
```

This is a good test, except remember that each time we run our test suite we will be creating this post. We need to make sure we delete this post before we run the test. So let's wrap that in a mongoose model `.remove()` method.


```js
var tour = { title: "post title", url: "https://www.google.com", summary: "post summary" }

Tour.findOneAndRemove(tour, function() {
  Tour.find(function(err, tours) {
    var tourCount = tours.count;
    chai.request('localhost:3000')
      .post('/tours', tour)
      .end(function (err, res){
        Tour.find(function(err, tours) {
          tourCount.should.be.equal(tours + 1);
          res.should.have.status(200);
        done();
      });
    });
  });
});
```

Now we have a test for the posts#create route that should be green. Can you make it fail? How about if our `post` object doesn't have a title, url, or summary? Those are all required fields. What do you see if you change that and run the test? Does it fail? How do you know what made it fail?

When a controller or route test runs, it runs itself and it hits your server endpoint code locally. That means you can put `console.log` or `debugger` statements in either one to check various values.
