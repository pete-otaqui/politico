Politico
========

Like all politicians, make a series of promises.  Unlike politicians, make sure
they are fulfilled.

## Why?

Because it's really annoying working in callback hell, and most of the options
are either too opinionated or too low-level.

Let's imagine I'm working Express and Mongoose, and for a "dashboard" page I
have to make quite a lot of calls to collect the data I want, e.g.

```javascript
get('/dashboard', function(req, res, next) {
  Expense.get({owner:req.user._id}, function(err, expenses) {
    if ( err ) return next(err);
    // note that we use the return value of one query for the next one:
    Scandal.get({expenses:expenses}, function(err, scandals) {
      if ( err ) return next(err);
      Moat.get({owner:req.user._id}, function(err, moats) {
        if ( err ) return next(err);
          Mortgage.get({owner:req.user._id}, function(err, mortgage) {
            if ( err ) return next(err);
          });
      });
    });
  });
});
```

Wouldn't it be much nicer to do this?

```javascript
var politico = require('politico');

get('/dashboard', function(req, res, next) {
  politico(
    Expense  .get({owner:req.user._id}),
    function(expenses) {
      Scandal.get({expenses:expenses})
    },
    Moat     .get({owner:req.user._id}),
    Mortgage .get({owner:req.user._id})
  ).then(
    function(results) {
      console.log(results); // an array of results
      res.render('dashboard', {
        expenses:  results[0],
        scandals:  results[1],
        moats:     results[2],
        mortgages: results[3]
      })
    },
    function(err) {
      return next(err);
    }
  )
});
```