---
layout: default
title: Service Container
parent: Architecture Concepts
---
# Introduction
The Laravel service container is a powerful tool for managing class dependencies and performing dependency injection. Dependency injection is a fancy phrase that essentially means this: class dependencies are "injected" into the class via the constructor or, in some cases, "setter" methods.

Let's look at a simple example:
```
<?php
 
namespace App\Http\Controllers;
 
use App\Http\Controllers\Controller;
use App\Repositories\UserRepository;
use App\Models\User;
 
class UserController extends Controller
{
    /**
     * The user repository implementation.
     *
     * @var UserRepository
     */
    protected $users;
 
    /**
     * Create a new controller instance.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
 
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        $user = $this->users->find($id);
 
        return view('user.profile', ['user' => $user]);
    }
}
```
In this example, the UserController needs to retrieve users from a data source. So, we will inject a service that is able to retrieve users. In this context, our UserRepository most likely uses Eloquent to retrieve user information from the database. However, since the repository is injected, we are able to easily swap it out with another implementation. We are also able to easily "mock", or create a dummy implementation of the UserRepository when testing our application.

A deep understanding of the Laravel service container is essential to building a powerful, large application, as well as for contributing to the Laravel core itself.
## Zero Configuration Resolution
If a class has no dependencies or only depends on other concrete classes (not interfaces), the container does not need to be instructed on how to resolve that class. For example, you may place the following code in your routes/web.php file:
```
<?php
 
class Service
{
    //
}
 
Route::get('/', function (Service $service) {
    die(get_class($service));
});
```
In this example, hitting your application's / route will automatically resolve the Service class and inject it into your route's handler. This is game changing. It means you can develop your application and take advantage of dependency injection without worrying about bloated configuration files.

Thankfully, many of the classes you will be writing when building a Laravel application automatically receive their dependencies via the container, including controllers, event listeners, middleware, and more. Additionally, you may type-hint dependencies in the handle method of queued jobs. Once you taste the power of automatic and zero configuration dependency injection it feels impossible to develop without it.
## When To Use The Container
Thanks to zero configuration resolution, you will often type-hint dependencies on routes, controllers, event listeners, and elsewhere without ever manually interacting with the container. For example, you might type-hint the Illuminate\Http\Request object on your route definition so that you can easily access the current request. Even though we never have to interact with the container to write this code, it is managing the injection of these dependencies behind the scenes:
```
use Illuminate\Http\Request;
 
Route::get('/', function (Request $request) {
    // ...
});
```
In many cases, thanks to automatic dependency injection and facades, you can build Laravel applications without ever manually binding or resolving anything from the container. So, when would you ever manually interact with the container? Let's examine two situations.

First, if you write a class that implements an interface and you wish to type-hint that interface on a route or class constructor, you must tell the container how to resolve that interface. Secondly, if you are writing a Laravel package that you plan to share with other Laravel developers, you may need to bind your package's services into the container.
# Binding
## Binding Basics
### Simple Bindings
Almost all of your service container bindings will be registered within service providers, so most of these examples will demonstrate using the container in that context.

Within a service provider, you always have access to the container via the $this->app property. We can register a binding using the bind method, passing the class or interface name that we wish to register along with a closure that returns an instance of the class:
```
use App\Services\Transistor;
use App\Services\PodcastParser;
 
$this->app->bind(Transistor::class, function ($app) {
    return new Transistor($app->make(PodcastParser::class));
});
```
Note that we receive the container itself as an argument to the resolver. We can then use the container to resolve sub-dependencies of the object we are building.

As mentioned, you will typically be interacting with the container within service providers; however, if you would like to interact with the container outside of a service provider, you may do so via the App facade:
```
use App\Services\Transistor;
use Illuminate\Support\Facades\App;
 
App::bind(Transistor::class, function ($app) {
    // ...
});
```

```
Tips: 
There is no need to bind classes into the container if they do not depend on any interfaces. 
The container does not need to be instructed on how to build these objects, 
since it can automatically resolve these objects using reflection.
```
## Binding Interfaces To Implementations
## Contextual Binding
## Binding Primitives
## Binding Typed Variadics
## Tagging
## Extending Bindings
# Resolving
## The Make Method
## Automatic Injection
# Method Invocation & Injection
# Container Events
# PSR-11
Laravel's service container implements the PSR-11 interface. Therefore, you may type-hint the PSR-11 container interface to obtain an instance of the Laravel container:
```
use App\Services\Transistor;
use Psr\Container\ContainerInterface;
 
Route::get('/', function (ContainerInterface $container) {
    $service = $container->get(Transistor::class);
 
    //
});
```
An exception is thrown if the given identifier can't be resolved. The exception will be an instance of Psr\Container\NotFoundExceptionInterface if the identifier was never bound. If the identifier was bound but was unable to be resolved, an instance of Psr\Container\ContainerExceptionInterface will be thrown.