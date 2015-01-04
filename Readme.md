# Express Resource Plus

Express Resource Plus is a fork of [express-resource new](https://github.com/tpeden/express-resource-new) providing a few more routes, refactors and an extra feature or two.

**** ONLY compatible with Express 4 ****

## Installation

npm:
    $ npm install express-resource-plus

## Usage

In your main application file (i.e. app.js or server.js) just add the following:

    var express = require('express'),
        Resource = require('express-resource-plus'), // <- Add this (Resource really isn't needed)
        app = express.createServer();

    app.configure(function(){
      // it is important to note that /app directory layout must follow a specific pattern to work properly see [App Layout](#app-layout)
      app.set('app_dir', __dirname + '/app');
      /* ... */
    });

### App Layout
  /app/:version/controllers/:controller_name/index.js
                routes.js


"What if I want to create a resource on the root path or change the id variable name or define middleware on specific actions?" express-resource-plus handles that by allowing you to set an `options` property on the controller object like so:

    var before = {
      auth : function(req, res, next) {
        // some auth logic
        next();
      },
      owner : function(req, res, next) {
        // resource owner logic
        next();
      }
    }

    module.exports = {
      options: {
        root: true, // Creates resource on the root path (overrides name)
        name: 'posts', // Overrides module name (folder name)
        id: 'id', // Overrides the default id from singular form of `name`
        before: { // Middleware support
          show: before.auth,
          update: [before.auth, before.owner],
          destroy: [before.auth, before.owner]
        }
      },
      index: function(request, response) {
        response.send('articles index');
      },
      /* ... */
    };

Lastly just call `app.resource()` with your controller name. Nesting is done by passing a function that can call `app.resource()` for each nested resource. Options can also be passed as the second parameter which override the options set in the controller itself.

    app.resource('articles', { controller : articles }, function() {
      app.resource('comments', { controller : comments });
    });

You can also create non-standard RESTful routes.

`./controllers/comments/index.js`:

    module.exports = {
      index: function(request, response) {
        response.send('comments index');
      },
      /* ... */
      search: function(request, response) {
        response.send("Search all comments on this article.");
      },
      reply: function(request, response) {
        response.send("Reply to comment: " + request.params.comment);
      }
    };

`./app.js`:

    /* ... */
    app.resource('articles', { controller : articles }, function() {
      this.resource('comments', { controller : comments }, function() {
        this.collection.get('search');
        this.member.get('reply');
      });
    });

## Default Action Mapping

Actions are, by default, mapped as shown below. These routs provide `req.params.article` for the substring where ":article" is and, in the case of the nested routes, `req.params.comment` for the substring where ":comment" is as shown below:

    articles:
    index    GET     /articles.:format?
    new      GET     /articles/new.:format?
    create   POST    /articles.:format?
    show     GET     /articles/:articleId.:format?
    edit     GET     /articles/:articleId/edit.:format?
    update   PUT     /articles/:articleId.:format?
    update   PUT     /articles.:format?  ( { where : { username : 'Steve' } })
    destroy  DELETE  /articles/:articleId.:format?
    destroy  DELETE  /articles.:format?  ( { where : { username : 'Steve' } })
    query    POST    /articles/query.:format?  ( { where : { username : 'Steve' } })
    describe HEAD    /articles

    article_comments:
    index   GET     /articles/:articleId/comments.:format?
    new     GET     /articles/:articleId/comments/new.:format?
    create  POST    /articles/:articleId/comments.:format?
    show    GET     /articles/:articleId/comments/:commentId.:format?
    edit    GET     /articles/:articleId/comments/:commentId/edit.:format?
    update  PUT     /articles/:articleId/comments/:commentId.:format?
    update  PUT     /articles/:articleId/comments.:format?  ( { where : { username : 'Steve' } })
    destroy DELETE  /articles/:articleId/comments/:commentId.:format?
    destroy DELETE  /articles/:articleId/comments.:format?  ( { where : { username : 'Steve' } })
    query   POST    /articles/:articleId/comments/query.:format?  ( { where : { username : 'Steve' } })


## Resource Loader
  You can now include  options.load (a middleware function) which will load the resource as a parameter in the request

## Content Negotiation

Content negotiation is currently only provided through the `req.params.format` property, allowing you to respond accordingly.

## License

    The MIT License

    Copyright (c) 2013 Simon Townsend <stowns3@gmail.com>

    Permission is hereby granted, free of charge, to any person obtaining
    a copy of this software and associated documentation files (the
    'Software'), to deal in the Software without restriction, including
    without limitation the rights to use, copy, modify, merge, publish,
    distribute, sublicense, and/or sell copies of the Software, and to
    permit persons to whom the Software is furnished to do so, subject to
    the following conditions:

    The above copyright notice and this permission notice shall be
    included in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
    EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
    MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
    IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
    CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
    TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
    SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
