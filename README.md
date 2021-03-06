# Express App Extended Boilerplate

[![Build Status](https://travis-ci.org/iximiuz/node-express-ex-boilerplate.svg)](https://travis-ci.org/iximiuz/node-express-ex-boilerplate)
 
## Features:
 - app object is Dependency Injection Container
 - asynchronous app initialization out of the box
 - service layer
 - non-http entry points (console, etc)
 - extremely testable architecture
 
## Intro
It's convenient to divide NodeJS modules into 3 types:

 - pure functional (provide pure function(s) without any state)
 - factories (provide constructors/factories functions)
 - executable
 
First two types of modules don't have any internal state. It's a very useful property, because 
such modules easy to test and use.
 
*Executable* modules do have an internal state (since the code in each NodeJS module is executed
only once during a first time `require` call of the module). However, if one has the same module
placed on two or more different paths, it's possible that the code of such module will be executed
as many times as it's required by different paths. Also, having a shared state withing the module
makes it less testable.
 
That is, it's very convenient to have modules only of first two types in your project. In ideal 
scenario, only *entry point* modules should be *executable*.
 
Most of the Express boilerplate projects are focused on having things as simple as possible.
They have pretty flat directory structure, don't consider an asynchronous app initialization 
code and have a lot of *executable* modules (all routes, app, logger, etc). It could be an 
appropriate choice for very small applications. However, having such structure for a long-living
project leads to a unsupportable code.
 
This boilerplate offers an *executable*-less architecture. All modules except *entry points* are
*factories* or *pure functional*. Even the routing system.
 
Another important feature of this boilerplate is a separated *service layer*. For more information
about it see **Example** section.

## Example 

    // services.js
    var monk = require('monk');
    var UserRepo = require('./dao/UserRepo');
    
    module.exports.register = function(app) {
        app.set('db', function(app, callback) {  
            // will be called only once
            var db = monk(app.options.mongo.dsn);
            callback(null, db);
        });
    
        app.set('userRepo', function(app, callback) {
            // will be called only once
            app.get('db', function(err, db) {
                callback(null, new UserRepo(db));
            });
        });
    };
    
    
    // route.js
    var Router = require('express').Router;
    
    var usersHandler = module.exports.usersHandler = function(app) {
        return function handler(req, res, next) {
            app.get('repo.user', function(err, userRepo) {
                userRepo.findAll(function(err, users) {
                    res.render('users', {users: users});
                });
            });
        }
    };
    
    module.exports.register = function(app) {
        var router = Router();
        router.get('/', usersHandler(app));
        app.expressApp.use('/users', router);
    };
    
    
    // app.js
    var services = require('./services');
    
    function Application(options) {
        ...
        services.register(this);
    }
    
    
    // web/app.js
    ...
    function WebApp(options) {
        ...
        for (var r in routes) {
            r.register(this);
        }
    }
        
    
    // www (entry point)    
    var config = require('../config');
    var WebApp = require('../web/app');

    var app = new WebApp(config);
    app.init(function(err) {
        if (!err) {
            app.startServer();
        }        
    });
    
## Usage
### Run web server
    npm run server_start  # default is localhost:3000
     
### Execute command line task
    npm run command ping
    npm run command users

## Documentation
The boilerplate provides an `Application` object, that is not an Express app.
This `Application` object is extended by `WebApp` object, which holds an Express app
object as a member. The `Application` object by itself can be used in *entry points*
which don't require *web* functional, for example `console` scripts.

Also the `Application` object is a Dependency Inject Container. 
One can use `app.set(name, service)` and `app.get(name, callback)` to register new
services and then access it within the route handlers or console scripts.

For more information about it see `Application::get()` and `Application::set()` JsDocs.
