worktalk
==========

A simple SailsJS project

##Step 1: Create your Sails project

###Create the worktalk directory, and cd into it.

```javacript
mkdir worktalk
cd worktalk
```

###Generate the sails files with the new command.

```javascript
sails new .
```

##Step 2: Generate the messages scaffolds
Sails.js auto-generates controllers and models for us when we use the `generate` command. Let's use that to create our messages structure.
* Create a message scaffold (`sails generate message`)
* Note how both the controller and the model were created (api/controllers/MessageController.js and api/models/Message.js)
* Use postman to POST some messages to your new endpoint. At this point, you can use any attributes you want, but let's post messages with this structure:

```javascript
{
  "body": "hello world",
  "from": "Don in Accounting"
}
```
* (Also, make sure your server is running)

##Step 3: Create a custom route and view
We have an auto-generated API, but let's create a simple view so we can see our data.
* Let's start by creating a controller called HomeController.

```javascript
sails generate controller home
```
* Open up the generated controller in api/controllers/HomeController.js
* Add an `index` method. Sails is built on express, and each of the controller's methods will expect a request and response parameter, just like Express does.
* Let's render a view.

```javascript
index: function(req, res) {
  res.view('home');
}
```
* Create a view file at views/home.ejs. Sails uses ejs as its templating engine.
* In the home.ejs file, let's add some code so we can see all the messages that have been displayed

```html
<div class="messages">
 <% messages.forEach(function(message) { %>
  <p><%=message.body%> (<%=message.from%>)</p>
 <% }); %>
</div>
```
* Notice the tags that are particular to ejs. These are obviously very different from the syntax that Angular uses.
* In order for our view to work, let's send the messages to the view function from our controller. Use the [`find`](http://sailsjs.org/#!documentation/models) method of the Message model, then use the [promise's](http://blog.mediumequalsmessage.com/promise-deferred-objects-in-javascript-pt1-theory-and-semantics) done method to render the view - it takes a function as an argument which you pass the possible error, and the successful return (in this case the messages).

```javascript
index: function(req, res) {
  Message.find().done(function(err, messages) {
    res.view('home', {messages: messages});
  });
}
```
* Finally, we need to override the default route and point it to your controller. Edit config/routes.js and point '/' to the index method of the HomeController.

```javascript
'/': 'HomeController.index'
```
* Test your new view and controller in the browser

##Step 4: Only allow authenticated workers to post messages
This has been fast and easy, but let's take it one step futher. You only want certain people to be able to post messages. Let's make a new policy file that will limit who can POST messages to our API.
* Create a file at api/policies/isEmployee.js
* Model your file after what is at api/policies/isAuthenticated.js (you can copy the text over to start with if you'd like)
* Notice the `next()` function. Just like with Express, this middleware allows us to preform some logic before a request is processed at the controller level. We can intervene if we don't like what the request contains (or doesn't contain).
* Let's use an if statement to check and see if the request has a query param named "employee_id." If it doesn't, let's return "forbidden." See the isAuthenticated policy file for reference.
* Add the policy to the config/policies.js

```javascript
  '*': true,
  MessageController: {
    '*': ['isEmployee'],
    'index': true
  }
```
* Now test POSTing messages. You now have to include an employee_id parameter in the query string in order to POST anything.

##Step 5 (Black Diamond): Show the messages in real-time
* Use the [real-time documentation](http://sailsjs.org/#!documentation/sockets) at Sails to show the incoming messages at home.ejs in real-time. The main tutorial video on the homepage of the Sails website also goes through this, and might be more helpful.

##Step 6 (Black Diamond): Integrate with Angular
Let's make the real-time communication part of an Angular service, and integrate Angular into your Sails app. 
You probably won't be able to use Yeoman very easily here, so you'll have to include the script tags for Angular as well as generate your own application module, your own controllers, and your own service. [This article](http://www.html5rocks.com/en/tutorials/frameworks/angular-websockets/) can help you with the Angular-to-socket communication, and poking around [this repo](https://github.com/levid/angular-sails-socketio-mongo-demo) can help you figure out how to organize and import your Angular files.
