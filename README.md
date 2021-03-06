# GeddyJs Ember REST Helper

### A fork of [geddy-rest](https://www.npmjs.com/package/geddy-rest) for [Ember](emberjs.com) Compatability.

## Differences

The only difference is how the `params` object is deserialized into the `options` passed into `Model` for `create` and `update` methods.

The normal version of `geddy-rest` passes the `params` directly into those methods.

This fork instead looks for a key on the `params` object that is a singular camel-cased name of the model, and passes its value as the options object.  This resembles the default `Ember` behavior.

## The normal README:

Use this simple library to avoid code repetition for Controllers REST actions. With one single line, you provide all the methods you need for your controller, in order to have a fully REST api.

It is made to be used with [GeddyJs](http://geddyjs.org/).

## Instalation

1. Run: `npm install --save geddy-rest`
2. Add the folowing line inside your controller:

  ```
  require('geddy-test').RESTify(this, MyModel);
  ```

3. Add the folowing line inside `/config/routes.js`:

  ```
  require('geddy-test').route(router, 'mymodel');
  ```

4. You are done, and you can access trough:
  ```
  GET /:model         ---> Find All
  GET /:model/:id     ---> Find First
  POST /:model/:id    ---> Create
  PUT /:model/:id     ---> Update
  DELETE /:model/:id  ---> Destroy
  ```

  _If you want `named` paths with `GET` method, read below._


## Nested findings

Imagine the folowing situation: `User` `hasMany` `Photos`. If you need to read the User photos, you would need to go to an endpoint like `GET /photos?userId=<theUserId>`, after getting the user information in `GET /users/<theUserId>`.

With that in mind, I made a simple helper that allows you to do even more with Geddy. This situation will become `GET /users/<theUserId>/photos`, in a really simple way.

We present you, nested associations with REST:

```
// Add this to your config/routes.js to allow use of nested findings
require('geddy-test').RestApi.route(router, 'users', {
  find: {
    // Enables Nested finding route
    nested: true
  }
});

// User Controller
// We are nesting the model method 'getPhotos' as 'photos'
require('geddy-test').RESTify(this, MyModel, {
  find: {
    nested: {
      photos: 'getPhotos'
    }
  }
});

// You can even nest multiple actions, and provide a custom callback like:
function specialMethod(cb){
  // Send back data to be replaced
  cb({custom: 'data', to: 'be', shown: true});
}

require('geddy-test').RESTify(this, MyModel, {
  find: {
    nested: {
      photos: 'getPhotos',
      friends: 'getFriends',
      address: 'getAddress',
      special: specialMethod
    }
  }
});
```

The example above would generate the folowing routes:
```
GET /users/:id/photos  -> find then getPhotos
GET /users/:id/friends -> find then getFriends
GET /users/:id/address -> find then getAddress
GET /users/:id/special -> find then run specialMethod
```

## Using it as a middleware

You can also use those methods for your own purpose. Look at some examples:

```
var Users = function (){
  // Inject methods into controller
  require('geddy-rest').RESTify(this, geddy.model.User);

  // Should route to: /users/:id/resume
  this.resume = function (req, res, params){

    // Execute custom find method
    this.find(req, res, params, function afterFind(err, model, cb){

      var data = {
        userName: model.name,
        userFriendsCount: model.getFriends().length
      };

      // Continue with it...
      return cb(data);

      // Or render it yourself
      respond(data);

    });
  }
}
```

## Advanced usage

- `RESTify(controller, model, opts)`

  Restify creates methods inside your given `controller`. It uses the `model` to _create/find/delete/update_.

  You can also assign some options:

  - `opts.[create|find|update|destroy]`: [`false` | `Object`]
    If set to false, will not generate the method.

    example:
    ```
    // Only enables find method
    RESTify(this, MyModel, {
      create: false,
      destroy: false,
      update: false,
    });
    ```

  - `opts.[create|find|update|destroy].action`: `String`
    Use this to change the desired method to be saved. You can, for example, generate it and use it the way you want:
    ```
    RESTify(this, MyModel, {
      find: {
        action: 'myFindAction'
      }
    });

    this.find = function (req, res, params){
      // Do whatever you want....
      var a = 2*9;
      // Delegate it to the REST find method
      this.myFindAction(req, res, params);
    }
    ```


  - `opts.[create|find|update|destroy].beforeRender`: `function`

    Use this property to receive actions just before rendering the data. You can either render it yourself (just don't call the `next` function), or process something before rendering and delegate it to the REST action egain:
    ```
    this._checkDbFirst = function (err, models, next){
      // You render the content here. Just don't call next.
      respond(models);

      // But if you do, pass the models to render
      next(models);
    }

    RESTify(this, MyModel, {
      create: {
        action: 'find',
        beforeRender: this._checkDbFirst
      }
    });
    ```

  - `opts.find.nested`: `Object`

    Put each nested method as the key, and the callback as the value. (Read above for more info);


  - GeddyJs is a well tought Framework. If you need to perform actions either `before` or `after` a call to the REST endpoint, use the methods `.before` and `.after`:
    ```
    // Calls prepareThings before find and create
    this.before(prepareThings, {only: ['find', 'create']});
    // Calls finishThingsUp after update and destroy
    this.after(finishThingsUp, {only: ['update', 'destroy']});
    ```

- `route(Router, ModelName, opts)`

  This will generate routes acordingly to your needs. It should be placed inside `/config/routes.js`.

  If you need to alter the `default routes`, or `HTTP methods` you can access it exacly like the `RESTify` method:
  ```
  // Change the default route of 'find' to 'get'
  // Change the destroy method to 'GET'
  // Disables update route
  route(router, MyModelName, {
    find: {
      action: 'get'
    },
    destroy: {
      method: 'GET'
    },
    update: false
  });
  ```

  One last usefull option, is the `opts.strict`, which defaults is set to `true`.

  If set to false, will generate PATHS to facilitate your life (during development?):
  ```
  GET /:model/find
  GET /:model/find/:id
  GET /:model/create
  GET /:model/destroy/:id
  GET /:model/update/:id
  ```
