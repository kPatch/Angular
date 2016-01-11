# Angular

A file structure should be self documenting and it should also lend itself to efficiency and ability to located resource
promote modular composition to extend your application indefinitely

We are going to develop in a feature-driven paradigm.
We are going to organize by feature not by type. We are not going to isolate our controller, our services etc..

Let's start off with the models because they are used accross the application

No worries on the tidious tasks - Gulp



Naming Conventions
We want the developers to know by the name of it exactly what it does
For example in bookmarks/edit/bookmarks-edit.js  

We have a module called 'categories.bookmarks.edit'


## Promoting Collections to models
- Extracting collection to services

## Working with $http and controllers
- We don't want our controllers to know the implementation details of how our data is being returned from the server (don't use 'resutl.data' in controller)
----- Look into AngularJS Architecture: Using $http to load JSON data

## AngularJS architecture: Control your promises with $q
- There are situation where you want to do something to that data before passing it to the controller (i.e. return a cache instead of performing a call).
THis is where the $q service comes in.

---- $q.when allows you to wrap a promise around an <object 


