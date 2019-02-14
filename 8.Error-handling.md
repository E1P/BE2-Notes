# Handling Errors

## Learning Objectives

- Learn about `400`, `404`, `405` and `422` error codes and how to handle them using express error-handling middleware functions.
- How to use the `router.all()` method in express.

## Error-handling

- What are the main sources of errors that we might encounter when developing our project?

### 400 - bad request

A 400 error means Bad Request, that is to say, the client did something wrong in their request. This is typically but not exclusively a bad ID format etc. If we have an end-point `api/films/:film_id` and a client makes a request of `api/films/abc` - this is clearly a bad request. Our ID should be an integer but instead we got an ID of "abc" which makes no sense at all!

Basically, it's when the user sends something malformed - either in the body or in the params.

### 404 - when the route does not exist!

This could be that the param ID is correctly formed but does not exist, or the route does not exist at all. There are usually two ways of deciding whether something is a 404 - whether the path does not exist at all, or whether, when you request it from the db, you get nothing back.

### 422 - unprocessable entity

When the entity conforms to the DBs expectations but cannot be processed for other reasons - i.e. unique key violations.

### 405 - method not allowed

When the route exists, but cannot be accessed with that method. I.e. doing a POST on `/articles/:article_id`

### route.all()

Middleware that is called on ALL other routes not specified. This is best for handling the method not allowed errors.

## Testing an error

```js
  describe("/api/films/:film_id",() => {

    it("GET status:400 responds with error message when request is made with a bad ID",() => {
      // assertion here
      return request.get("/api/films/abc")
      .expect(400)
      .then(res => {
        expect(res.body.message).to.equal("Bad Request, incorrect form for film_id!")
      })
    });
  })
});

```

- At the moment this test will fail as we haven't set up a controller : however, do we even need a controller middleware function to handle such an error ?

### Error Handling Blocks

Unlike pgp, knex throws SQL error codes out as the errors. Which is useful, because they are consistent, and have definitions which provide useful feedback to our api users. You can also use a reference object as a way of checking that, when an error is passed to `next`, it conforms to its expectations.

Like so:

```js
exports.handle400 = (err, req, res, next) => {
  const codes = {
    23502: 'violates not null violation',
    22P02: 'invalid input syntax for type integer',
  };
  if (codes[err.code]) res.status(400).send({ msg: codes[err.code] });
  else next(err);
};
```

- By doing this check in one place - it saves us from checking for 400s or 404s all the time in lots of different controller middleware.

### Some hints:

- One problem you may face, is that you cannot delete something like an `article` easily. Why? Because other entries in other DBs use its ID as a foreign key. In this case, you might want to add this to your migrations, to save you from sending 2 delete requests:

```js
wizardsTable
  .integer('house_id')
  .references('houses.house_id')
  .unsigned()
  .onDelete('CASCADE');
```

- One annoying thing about `knex`, is that it always returns results - even when you expect one thing back, in an array. You should destructure these arrays to make sure you're sending back an object, rather than an array with a single item.
