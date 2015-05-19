## 1 Introduction
 
In this article we are going to explore two popular MV* JavaScript frameworks: AngularJS and Ember.js. This time we are putting them head-to-head and will compare their advantages and most common problems interacting with a Rails-powered backend. By the end of this article we will have enough information to decide, depending on our web project requirements, whether to choose Angular or Ember.
 
Ruby on Rails is a pretty useful framework for developing cutting-edge web applications; it includes all the necessary tools to help the programmer deliver tangible results in a fast, well-structured and maintainable way, going from simple prototyping to a completely functional solution without too many complications. But there are situations where we want to offer our users a better or more interactive experience using our application; here is where the client-side MV* frameworks acquire great importance.   

It's worth making one thing clear: we are not saying that Rails isn't up to the challenge. It's just that given its default set of tools (in this case Javascript/jQuery) it would take more time and effort to achieve our goal, and if we are talking about agile software development, everything is about delivering results (value) in less time, right?

## 2 Single-page applications and 'modules'

When we talk about MV* client side-appications, we can classify them under two main categories - single-page applications and "modules."

Single-page applications make extensive use of Javascript. When users visit an initial url (e.g. myapplication.com/index), the backend serves a single view with all the necessary resources to start a fully-featured Javascript application. From that point on, the backend won't serve HTML views, but will instead interact with the client application as an API (commonly in JSON format). Here, the Javascript application takes control of the information flow, handling routes and dispatching all the content by itself.

On the other hand, we have Rails applications that require exhaustive use of Javascript to improve the user experience only in some particular sections; in this case, rendering a particular controller/action might trigger a more fully-featured user interface, but navigating to another section will return to the usual MVC Rails flow. I myself haven't found a standard name to call these applications, so we will roughly call these applications 'Modules.'

Going forward, we will distinguish between single-page applications and modules only if either type receives a particular advantage or disadvantage in a specific situation.

## 3 Installation process

Before jumping into the interaction details between our Rails backend and our frontend Javascript application, choosing an appropriate installation method &mdash; ideally one which lets us handle our dependencies in order and facilitates updates &mdash; is required.

The easiest and most-recommended way of incorporating Javascript frameworks is through gems; gems are automatically integrated into the assets pipeline, reducing the assets' import errors to almost 0. It's worth mentioning that relying on gems makes us 100% dependent on the the gem's maintainer keeping it updated.

### 3.1 Ember

The solution here is pretty simple: the Ember team has developed its own gem, which integrates with Rails 3.1 or newer, called ember-rails. It also includes some generator commands. The gem boils down the installation process to the following steps:

1. Add the gem to your Gemfile

<!--code lang=ruby linenums=true-->
  
    gem 'ember-rails'
    gem 'ember-source'

2. Execute `bundle install`

3. (Optional) Execute the generator

<!--code lang=ruby linenums=true-->
  
    rails generate ember:bootstrap

### 3.2 Angular

Decisions. Why do they always make us make decisions? As there is no "official" gem to integrate Angular with Rails, we have to choose between two main options: [angular-rails](https://github.com/ludicast/angular-rails) and [angularjs-rails](https://github.com/hiravgandhi/angularjs-rails). 

We would think angular-rails is the best option because, similar to ember-rails, it includes not only the assets pipeline integration, but also generators. The problem here is that the gem hasn't been updated in more than two years and is stuck on Angular 1.0.2; in this situation, angularjs-rails is a better choice. The gem is installed this way:

1. Add the gem to your Gemfile

<!--code lang=ruby linenums=true-->
    
    gem 'angularjs-rails'

2. Execute `bundle install`


## 4 Project setup

Now that we have our favourite gem installed in our project, we can proceed to generate the frontend application's instance. If this application instance is visible through any Javascript file inside `app/assets/javascript`, we can generate virtually any folder structure for our Javascript application; each framework solves the issue its own way.

### 4.1 Ember

Ember requires storing the application's instance for further use; the usual aproach is initiating the application in the main Javascript manifest file `application.js`.

<!--code lang=javascript linenums=true-->

    window.App = Ember.Application.create()

App is usually the standard name for an Ember application's instance (if we have executed the generator during the instalation process, this App variable would be atutomatically generated). Ember inherits much of Rails' ideology, and in this case you can feel the "convention over configuration."

The variable that contains our Ember application _should_ be accessible through all of our Javascript code; the most common practice is to include it in the `window` object. This will let us have an ordered structure inside the `assets/javascripts` folder, otherwise we would be forced to include _all_ of our Ember code (controllers, views, etc.) in a single file. Those practices were left behind in 2000.

### 4.2 Angular

On the other hand, Angular uses modules to handle application instances. We can have many modules, and Angular will always be able to find them by name, without the need for storing then in global variables. We can initialize our Angular module in `application.js`:

<!--code lang=javascript linenums=true-->
  
    angular.module('demoApp', [])

But what is that second parameter? Why an empty array? Angular makes use of a concept called dependency injection to enhance our modules; in this case, Angular expects us to send an array with all the required dependencies, and we are injecting 0 dependencies.

<!--code lang=javascript linenums=true-->

    angular.module('demoApp')

If we don't provide the second parameter, Angular will search for the instance of `demoApp`. Maybe this could be interpreted as a design flaw, but in reality we are differentiating a getter from a setter. This way we can have an ordered folder structure without worrying about losing our modules' instances.

## 5 Turbolinks

Turbolinks is a new feature included in Rails 4 with the main goal of accelerating the browser's loading speed whenever a link inside our application is visited. Basically, it holds the current page's instance between requests and only replaces the `<body>`, so the browser doesn't have to recompile Javascripts and CSS.

This looks like a really advantageous feature for our Rails applications (Turbolinks' GitHub page mentions making it up to twice as fast), but in the case of MV* applications we have some details to keep in mind.

1. Our application is very unlikely to require Turbolinks' help for increasing communication speed with our server; by its nature, communication is usually made with JSON objects and JSON objects representing `users` are lighter than an HTML users table.

<!--code lang=json linenums=true-->

        users: [
          { id: 1, first_name: 'John', last_name: 'Doe' },
          { id: 2, first_name: 'Jane', last_name: 'Doe' },
        ]


<!--code lang=markup linenums=true-->
  
    <table>
      <tr>
        <td>1</td>
        <td>John</td>
        <td>Doe</td>
      </tr>
      <tr>
        <td>2</td>
        <td>Jane</td>
        <td>Doe</td>
      </tr>
    </table>
    
2. Our MV* applications require some sort of notice to indicate everything is ready to start (for instance, the `ready` event). If we let turbolinks handle our application's links navigation, these events are substituted with Turbolinks' events. In other words, `ready` will be triggered only once on our application's first render (usually visiting the root url `/`), from there on, `page:load` will be triggered instead. Putting this in code, we would have the following behaviour:

<!--code lang=javascript linenums=true-->

    $(document).on 'ready', ->
      console.log("I'm ready!"); // prints once, when page is initially loaded

<!--code lang=javascript linenums=true-->

    $(document).on 'page:load', ->
      console.log("I'm loaded!"); // prints everytime a page is loaded after following a link


With this information, we might think we are going to have compatibility issues between Turbolinks and our client-side application, and guess what... you're right!

### 5.1 Ember

#### 5.1.1 Single-page application <a name="remove-turbolinks"></a>

As we mentioned before, a SPA uses the Rails backend to initialize itself, and from there on, all the flow is controlled by the client-side application, hitting the Rails backend just to query information in JSON format. In this case, because of the conflicts generated with Ember, Turbolinks will lead to more problems than benefits. The Ember community suggests removing Turbolinks from our Rails project with the following steps:

1. Remove Turbolinks from the Gemfile

<!--code lang=ruby linenums=true-->  

    gem 'turbolinks'

2. Remove Turbolinks form our `application.js`

<!--code lang=javascript linenums=true-->

    //= require turbolinks

3. Remove `data-turbolinks-track => true` reference from our layout

<!--code lang=ruby linenums=true-->

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>

This will completely remove Turbolinks from our project, letting Ember do its job without complications.

#### 5.1.2 Module application

The situation here turns out to be more complex if, for instance, our application handles three resources &mdash; _users_, _profile_ and _sites_ &mdash; and we only want Ember to handle the _users_ resource. Removing Turbolinks might not be a good idea; we don't want to sacrifice navigation speed for our whole application in favour of bringing improved user experience for a single module/resource, do we?

It turns out that with Ember there is not much to do. When using Ember, the tendency seems to be building a SPA or not using Ember at all &mdash; there are some brave people out there that managed to hack their way through this situation with a solution similar to Angular's (we can read about Angular's solution some more lines below). Listening to some of Turbolinks' load events and invoking the `reset` method from the Ember instance, is something like this:

<!--code lang=javascript linenums=true-->
  
    $(docunemt).on 'page:load', ->
      App.reset();

But __this is far from being a recommendation__. The `reset` method is used only in automated test environments, where we need to restart our Ember instance between tests. If we take a look at [Ember's reset method docs](http://emberjs.com/api/classes/Ember.Application.html#method_reset) we can read the following text:

> __reset__: Reset the application. This is typically used only in tests.

### 5.2 Angular

#### 5.2.1 Single-page application

Similar to Ember's case, if our application will use Rails only as an API, it doesn't make any sense to keep Turbolinks around. We can follow the same [previous procedure](#remove-turbolinks) to remove Turbolinks from our application.

#### 5.2.2 Module application

The issue with Angular applications under the 'Module' approach is similar to Ember applications, but unlike Ember, Angular provides us with a mechanism to initialize our applications manually; even Angular's guide informs us about this possibility in the [Bootstrap](https://docs.angularjs.org/guide/bootstrap) section.

> __Bootstrap__: This page explains the Angular initialization process and how you can manually initialize Angular if necessary.

Good! It looks like we got it covered. The trick is to use the _ng-app_ directive and handle Turbolinks' page load events. The approach that has worked for me is:

1. Find al HTML tags containing a _ng-app_ directive

2. Apply the _bootstrap_ method in each of those elements __only__ when the event `page:load` gets triggered

In code, this boils down to:

<!--code lang=javascript linenums=true-->

    ready = ->
      $('[ng-app]').each -> json.usuarios do |json|>
        module = $(this).attr('ng-app')
        angular.bootstrap(this, [module])

    $(document).on 'page:load', ready

It's worth mentioning that we should only initialize the application by hand when we visit a URL via Turbolinks. If we initialize the application manually on `ready`, Angular might send an error message stating that the application has already been initialized, or we would even end up with two different instances of the same application/controller.

## 6 CSRF token

Once we have our MV* application up and running, the time comes to interact with our Rails backend. As long as we limit ourselves to query information, everything will be OK. Unfortunately, this will never be the case; the idea of having a MV* client-side application is to query and update information to and from the backend.

Since version 3, Rails has included CSRF token verifications in order to avoid _Cross Site Request Forgery_ attacks, but starting from Rails 4, this misbehaviour is __completely blocked__, returning an error 422 `:unprocessable_entity` to our client (or attacker).

<!--code lang=bash linenums=true-->

    Started POST "/users.json"
    Can't verify CSRF token authenticity
    Completed 422 Unprocessable Entity in 1ms

The CSRF token requested by Rails can be found in the HTML document's header; no matter if we are talking about SPAs or Modules, the meta tag will be present:

<!--code lang=markup linenums=true-->
    
    <meta content="4EbcUD9tnb+ATyqDG78FTQoBNImHR104IhDfccisHpc=" name="csrf-token" />

Let's see how each framework manages to solve this security issue.

### 6.1 Ember

Usually, we would have to read the CSRF token from the document's header and then manipulate AJAX interactions to include it in the header of every AJAX call prior to its execution. Here we have a jQuery Example:

<!--code lang=javascript linenums=true-->

    # read CSRF token
    token = $("meta[name=\"csrf-token\"]").attr("content")

    # include token in request header
    $.ajaxPrefilter (options, originalOptions, xhr) ->
      xhr.setRequestHeader "X-CSRF-Token", token

Fortunately for Ember (which depends on jQuery), __Rails has already implemented this exact task__ through its unobtrusive Javascript adapter [jquery-ujs](https://github.com/rails/jquery-ujs), so we just have to ensure that we have the adapter included in our `application.js` and everything should work just fine.

Here I'm including a link to the [CSRFProtection method definition in jquery-ujs](https://github.com/rails/jquery-ujs/blob/b2787d457f2680b67103f611fa91f95fe8509eb2/src/rails.js#L57-L61) available at the time we published this article if any reader is interested.

### 6.2 Angular

Because there are no Angular gems that implement as much Rails integration as with Ember, we need to add a mechanism to include the CSRF token similar to the one implemented by the _jquery-ujs_ driver, but applied to Angular's backend communication services.

#### 6.2.1 Server-side solution

By itself, the solution we are going to offer for consideration could not be considered as a solution; it's more like a dirty fix, but we are mentioning it for the sake of completeness. This fix consists of __deactivating__ the CSRF token validation from the Rails' controllers that are consulted by the MV* application.

<!--code lang=ruby linenums=true-->

    # users_controller.rb

    class UsersController < ApplicationController
      skip_before_filter :verify_authenticity_token
      ...

I recommend avoiding this solution, because by doing this we are removing a security check, leaving an open door in our backend.

#### 6.2.2 Client-side solution

The client-side solution is a cleaner and also more correct approach; instead of removing the server-side requirement, it complies with it, including this security token in the messages sent to the server.

Angular provides many ways to communicate with a backend, which include the `$http` and `$resource` services; we can use the `$httpProvider` service to alter the behaviour of both services. In this case, the solution is to make the service adjustments inside our Angular application's configuration, so we can set the communication headers globally.

<!--code lang=javascript linenums=true-->

    angular.module('demoApp').config ($httpProvider) ->
      # read CSRF token
      token = $("meta[name=\"csrf-token\"]").attr("content")

      # include token in $httpProvider default headers
      $httpProvider.defaults.headers.common['X-CSRF-TOKEN'] = token

## 7 Consuming a REST API

We've arrived to the most important part regarding an MV* client-side application, communicating with our backend and executing (at least) CRUD actions on our server's available resources. By default, when we generate a scaffold for a Rails resource, we should have all the controller methods required to execute CRUD actions already; also JSON formatted answers and `json.builder` templates are ready to be used in order to establish proper communication with our client application.

### 7.1 Ember

From the client's point of view, sending information to a Rails backend has no complications. The usual way of doing this is through the Ember Data's RESTAdapter, which implements all the necessary REST idioms to interact with our Rails backend.

<!--code lang=javascript linenums=true-->

    App.User = DS.Model.extend
      first_name: DS.attr 'string'
      last_name: DS.attr 'string'

The problem arises when we try to process/interpret the returning data. The data serializer used by RESTAdapter depends on a root node to be included in the server's response messages in order to understand them correctly.

> Error while processing route: users No model was found for '0'

Maybe this error message isn't clear enough, but now that we know Ember requires the root node, we can look inside the server's reponse in search of answers.

<!--code lang=javascript linenums=true-->

    [
      { id: 1, first_name: 'John', last_name: 'Doe' },
      { id: 2, first_name: 'Jane', last_name: 'Doe' },
    ]

Solving this is not that hard, but unfortunately it involves making server-side adjustments. Rails has already generated a couple of `json.builder` templates to format answers in case of datasets (`index.json.jbuilder`) and single objects (`show.json.jbuilder`). If we adjust these two `jbuilder` templates to include the root node, everything should work as expected.

<!--code lang=javascript linenums=true-->

    # wrap existing template content with json.user block
    json.user do |json|
      ...
    end

### 7.2 Angular

Happily for Angular, no matter if we use _$http_ or _$resource_ service, the response sent by Rails comes in a format easily handled by Angular. The `PUT` idiom requires some of our attention, and the following examples will cover both services.

For the _$http_ service, using the `.put` shortcut and wrapping the data within a key named after the Rails resource we are interacting with would be enough, `user` in this case:

<!--code lang=ruby linenums=true-->

    $http.put("/user/#{ user.id }.json", { user: user }).success (data) ->
      console.log 'updated!'

For the _$resource_ service, we just need to specify that the updates are going to be made with the `PUT` method. The following snippet shows the implementation:

<!--code lang=javascript linenums=true-->

    $resource '/users/:id.json',
    id: '@id'
    ,
    update: 
    method: 'PUT'

This way, `User` objects instantiated from `$resource` will send a `PUT` petition instead of the usual `POSt` when updating.

## 8 JS minification

We have covered the most common integration points with our Rails endpoint, we have our Javascript application up and running in development, and we are eager to jump to production. Rails, beneath the assets pipeline optimizations, runs a Javascript code minification, reducing variables' names and eliminating whitespaces. But why are we interested in this procedure? Does Javascript minification affect our application's proper behaviour?

### 8.1 Ember

Ember doesn't have too much to worry about; the minification process is standard, all variables' names and their conversions will be consistent, and the jump to production won't be complicated. Our minified Ember application's code should work as in development.

### 8.2 Angular

Angular is a different story. Angular implements, throughout the framework, something called dependency injection as a mechanism to incorporate functionality in almost any component inside our application. If we want our controller to use a service, we simply inject it:

<!--code lang=javascript linenums=true-->

    angular.module('demoApp').controller 'demoCtrl', ($scope, $http) ->
      ...

This is a really cool Angular feature. But, when code minification takes place, the name of the `$scope` service will be reduced to, lets say `$s`, which Angular knows nothing about and hence will throw an `Uncaught Error: Unknown provider` error message. This error message isn't too useful, but we can apply solutions in both server and client side.

#### 8.2.1 Client-side solution

It's not that a rule has been written, but the tendency in web applications is that the client adapts to the server's requirements and not vice versa. Under this precept, we can adjust our Angular application so it won't be affected by Javascript minification. For our example `demoCtrl` controller, we can rewrite it with inline notation as follows:

<!--code lang=javascript linenums=true-->

    angular.module('demoApp').controller('demoCtrl', ['$scope', '$http', ($scope, $http) ->
      ...
    ])

Maybe this notation is not so clear; but hey, it will save us some headaches and head-scratching when going live to production. If you've just started your Angular project, use this syntax from the beginning. If you already have an existing project, I recommend spending some time to rewrite your controllers, factories, and other declarations with this syntax. In the end, you have tests that will tell you if something gets broken, right?

#### 8.2.2 Server-side solution

For those cases where adjusting the method definitions is not an option (maybe your project is too big or there aren't tests), there's an option to adjust your Rails server, specifically the Javascript's minification method. In our production configuration file `production.rb` we make the next adjustment:

<!--code lang=ruby linenums=true-->
    # config/environments/production.rb

    config.assets.js_compressor = Uglifier.new(mangle: false)

This will minify the Javascript code, without altering variable names. The next time we deploy our project to production, our Angular application should work correctly.

It's worth mentionting that this solution is not recommended, because while preserving variable names, we get a not-optimal Javascript minification.

## 9 Summary

We have looked at many integration points between a Rails backend and an Ember/Angular frontend. In some cases, the integration with Ember turns out to be easier; in other cases, Angular seems to fit better.

It appears that if we are dealing with a single module which should bring an enhanced user experience, maybe it doesn't make much sense to integrate Ember and an Angular application would be enough.

In the case of SPAs, Ember's configuration/integration step makes more sense as it will pay off in the future. But please don't misinterpret: Angular is also up to the challenge of building large-scale SPAs, and its configuration/integration with a Rails application is equally complex no matter the size.

At the end of the day, these integration details plus our knowledge regarding both MV* frameworks by themselves will help us make the right choice for our project. If you have any questions, be sure to book me for an AirPair!
