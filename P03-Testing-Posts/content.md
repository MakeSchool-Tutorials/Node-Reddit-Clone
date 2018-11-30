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
const chai = require("chai");
const chaiHttp = require("chai-http");
// you could also require your server.js file there
// const server = require('../server.js')
const should = chai.should();
// give the option to use expect() not just should
const expect = chai.expect;

chai.use(chaiHttp);

describe("site", () => {
  // Describe what you are testing
  it("Should have home page", (done) => { // highly suggested not to use
    // Describe what should happen
    // In this case we test that the home page loads
    chai.request("localhost:3000")
      // instead of using the hardcoded route pass the server const in
      //.request(server)
      .get("/")
      .then((res) => {
          // look at the magic of what is in store and what to look for
          //console.log(res)
          res.should.be.equal(200); // start off with a simple test
          return done()
      })
      .catch((err) => {
          return done(err)
      })
      // .end((err, res) => { // best to use promises .then().catch()
      //   if (err) {
      //     return done(err);
      //   }
      //   res.status.should.be.equal(200);
      //   return done(); // Call done if the test completed successfully.
      // });
  });
});
```

This test tests that the response's status should be equal to 200 - which if you recall your HTTP status codes, means the response is successful.

Now let's run the test. First update your `package.js` file to have a test command:

It is best to use the new ones stated below `"test-watch": "nodemon --exec 'npm test'"` to continuously see if your test are running without worrying to exit out of the server and restarting it.
```json
"scripts": {
  "test": "mocha",
  "test-watch": "nodemon --exec 'npm test'"
},
```

In order for this test to run the server will have to be running on localhost 3000. To do this you will need to have two terminal windows open. Run your server in one then run the script below in the other.

Now we can run our tests with:

```bash
$ npm test
```

What was the result? Can you make the test fail?

# Testing Posts#Create

Next let's make a test for the posts#create route we made. We can make a new file in `test` called `posts.js`.

```js
...
// Import the Post model from our models folder so we
// we can use it in our tests.
const Post = require("../models/post");
...

describe("Posts", () => {
  it("should create with valid attributes at POST /posts", done => {
    // TODO: test code goes here!
  });
});
```

The order of pseudocode we want to see is as follows

```js
// How many posts are there now?
// Make a request to create another
// Check that the database has one more post in it
// Check that the response is a successful
```

So if we write that in:

We should create a prior function to delete a lot of inputs into our database

```js

// test/seed/seed.js
// create a folder named seed then create a file named seed.js within the seed folder
// seed stands for seed data
const Post = require('./../../models/post');

const posts = [{
  title: 'Test post for Reddit',
  url: 'https://makeschool.com',
  summary: 'A brief history...'
}, {
  title: 'Not another bad test',
  url: 'https://github.com/jayceazua/',
  summary: 'Hopefully you learn something'
}]; // we will later create a messed up post where we missed required inputs

const populatePosts = (done) => {
    Post.deleteMany({}).then(() => {
        let postOne = new Post(reviews[0]).save();
        let postTwo = new Post(reviews[1]).save();
        // Promise all method waits for all promises to resolve.
        return Promise.all([postOne, postTwo])
    }).then(() => done());
}

module.exports = {
    posts,
    populatePosts
}

```

```js

...
const Post = require("../models/post");
const {posts, populatePosts} = require('./seed/seed.js');
...

describe("Posts", () => {
    beforeEach(populatePosts) // populate seed data for posts

    // INDEX
    it('should index ALL posts on / GET', (done) => {
        Post.find({}).then((_posts) => {
            chai.request(server)
                .get('/')
                .then((res) => {
                    // start off with a basic test
                    expect(res).to.have.status(200);
                    expect(res).to.have.header('content-type', "text/html; charset=utf-8");
                    // we test with text instead of body because content-type is "text/html"
                    expect(res.text).to.have.string(`${_posts[0]._id}`);
                    expect(res.text).to.have.string(`${_posts[1]._id}`);
                    expect(res.text).to.have.string(`${_posts[1].movieTitle}`);
                    expect(res.text).to.have.string(`${_posts[0].movieTitle}`);
                    expect(res.text).to.have.string(`${_posts[0].title}`);
                    expect(res.text).to.have.string(`${_posts[1].title}`);
                    return done()
                })
                .catch(err => done(err))
        }).catch(err => err)
    });
    // CREATE
    it('should CREATE a new post on / POST', (done) => {
        const demoPost = ({
            title: 'This is a test for the sake of testing!',
            url: 'https://www.chaijs.com/api/bdd/',
            summary: 'Do you believe in the power of testing?'
        });
        chai.request(server)
            .post('/posts')
            .send(demoPost)
            .then((res) => {
                expect(res).to.have.status(200); // basic test
                expect(res).to.be.html; // basic test
                Post.findOne({ title: demoPost.title }).then((post) => { // complex test
                    expect(demoPost.title).to.equal(post.title);
                    // need to find the proper way of testing redirecting
                    expect(res).to.redirect;
                    expect(res.redirects[0]).to.include(review._id); // makes sure the redirect url includes the Id
                    expect(res.req.path).to.not.equal(`${app.mountpath}`); // makes sure it redirected
                }).catch(e => e);
                // console.log(res.body)
                return done();
            })
            .catch(e => done(e));
    });

    // think of other RESTful routes you are able to test
    // READ
    // UPDATE
    // DELETE
});
```

This is a good test, except remember that each time we run our test suite we will be creating this post. We need to make sure we delete this post before we run the test. So let's wrap that in a mongoose model `.remove()` method.



```js
// better delete test...
it('should delete a SINGLE post on /posts/<id> DELETE', (done) => {
    Post.find({}).then((data) => {
        let reviewId = String(data[0]._id)
        expect(data.length).to.equal(2);
        chai.request(server)
            .delete(`/posts/${reviewId}`) // deleting the review from index [0]
            .then((res) => {
                expect(res).to.have.status(200)
                Post.find({}).then((_posts) => {
                    expect(_posts.length).to.equal(1);
                    expect(data[0]).to.not.equal(_posts[0]);
                }).catch(e => e);
                expect(res).to.redirect;
                return done();
            })
            .catch(e => done(e));
    }).catch(e => e);
});
```

Now we have a test for the posts#create route that should be green. Can you make it fail? How about if our `post` object doesn't have a title, url, or summary? Those are all required fields. What do you see if you change that and run the test? Does it fail? How do you know what made it fail?

When a controller or route test runs, it runs itself and it hits your server endpoint code locally. That means you can put `console.log` or `debugger` statements in either one to check various values.
