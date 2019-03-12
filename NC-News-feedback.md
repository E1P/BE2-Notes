  37 passing (8s)
  17 failing

  1) /
       GET status:404 for any non-existent route:
     Error: expected 404 "Not Found", got 405 "Method Not Allowed"

* Any non-existent GET request should respond with 404!  At the moment, the request and response are being passed to your final `handle405` error handler

  2) /
       /api
         /articles
           (CORE TESTS) --> GET status:200
             responds with an empty array for articles queried with non-existent topic:
     Error: expected 200 "OK", got 404 "Not Found"

* If you request articles for a given topic and the topic exists but there are no associated articles for that topic, then this is not 404.  You can simply serve up an empty array and respond with a 200

  3) /
       /api
         /articles
           (CORE TESTS) --> GET status:200
             will ignore an invalid sort_by query:
     Error: expected 200 "OK", got 400 "Bad Request"

* Your controller should be ignoring an invalid sort_by query.  If you `?sort_by=frogs` then this should not respond with 400 - merely a 200

  4) /
       /api
         /articles/:article_id
           PATCH status:200 and an updated article when given a body including a valid "inc_votes" (VOTE UP):
     Error: expected 200 "OK", got 400 "Bad Request"

* At the moment you most likely have a condition in your articles controller that is stopping the patch logic from happening.
`typeof req.body.inc_vote !== 'number'` this boolean condition is always true ! as req.body.inc_vote is always a string

  5) /
       /api
         /articles/:article_id
           PATCH status:200 responds with an updated article when given a body including a valid "inc_votes" (VOTE DOWN):
     Error: expected 200 "OK", got 400 "Bad Request"

* Same issue here!

  6) /
       /api
         /articles/:article_id
           PATCH status:200s no body responds with an unmodified article:
     Error: expected 200 "OK", got 400 "Bad Request"

* Premature validation is again almost certainly causing this issue -  though not explicit in the README we said that for no requst body it should just be 200 and leave the article unmodified: however, if you want to treat this is as 400, then that is also OK

  7) /
       /api
         /articles/:article_id
           DELETE responds with a 204 when deleting an article without comments (no comments required to perform delete):
     Error: expected 204 "No Content", got 404 "Not Found"

* no leftJoin in `fetchArticlesById()`

  8) /
       /api
         /api/articles/:article_id/comments
           GET
             (CORE TESTS) --> status:200
               responds with an array of comment objects:

      AssertionError: expected { Object (comment_id, author, ...) } to have keys 'votes', 'comment_id', 'body', 'created_at', and 'author'
      + expected - actual

       [
      -  "article_id"
         "author"
         "body"
         "comment_id"
         "created_at"
      
* Not the end of the world this one! -> the comments don't need article_id but not too bad that they do

  9) /
       /api
         /api/articles/:article_id/comments
           GET
             (CORE TESTS) --> status:200
               can be sorted by author (DEFAULT order=desc):

      AssertionError: expected 'butter_bridge' to equal 'icellusedkars'
      + expected - actual

      -butter_bridge
      +icellusedkars
      
      at request.get.expect.then (spec/nc-spec.js:307:52)

* Don't do the initial query for comments by an article_id

  10) /
       /api
         /api/articles/:article_id/comments
           GET
             (CORE TESTS) --> status:200
               can be sorted by votes (DEFAULT order=desc):

      AssertionError: expected 14 to equal 100
      + expected - actual

      -14
      +100
      
      at request.get.expect.then (spec/nc-spec.js:315:51)

You are de-structuring sortBy and not `sort_by` in your controllers

  11) /
       /api
         /api/articles/:article_id/comments
           POST
             POST responds with a 422 when given a non-existent username:
     Error: expected 422 "Unprocessable Entity", got 500 "Internal Server Error"

* No error-handlers are catching this problem at the moment

  12) /
       /api
         comments/:comment_id
           PATCH status:200 and an updated comment when given a body including a valid "inc_votes" (VOTE DOWN):
     Error: expected 200 "OK", got 400 "Bad Request"

* Looks like you are putting in certain checks here that are catching this request and treating it as though it is an error.

  13) /
       /api
         comments/:comment_id
           PATCH status:200 with no body responds with an unmodified comment:
     Error: expected 200 "OK", got 400 "Bad Request"

* Same issue here!

  14) /
       /api
         comments/:comment_id
           PATCH status:404 non-existent comment_id is used:
     Error: expected 404 "Not Found", got 400 "Bad Request"

* You are hitting one of the 400 conditions before you even get to a 400 check



  16) /
       /users/:username
         GET status:200 responds with a user object when given a valid username:
     Error: expected 200 "OK", got 405 "Method Not Allowed"

* No router for this end-point at the moment

  17) /

       /users/:username
         GET status:404 when a non-existent username:
     Error: expected 404 "Not Found", got 405 "Method Not Allowed"

* No router this endpoint at the moment so just 404 given


## General

* Remove all of your superfluous console.logs!
* Not sure why you have to do `checkTopicExists()` in your articles-controller - surely you can make the request and knex will tell you if there is no author, for example
* Also `checkTopicExists()` is mis-leading as it is not actually checking but making a query for all the articles of a given topic
* You could also think about implementing `getArticles` with a call to `Promise.all()` as well
* Don't need the final `.then()` block at the end of the promise chain in your `seed` function
