# File Structure

- Should be self-documenting and should lend itself to find resources very quickly
- Should promote modular composition to extends files
- We'll organize by feature
-- Makes easy to test by feasture
- Common features should go in a 'common' directory. Feature that are shared accross components
For example

# Sub-Modules
Organize Code into containers

- Models
-- i.e. eggly.models.bookmarks, eggly.models.categories , categories.bookmarks.edit



# Navigating Unique State of Our application using $stateproviders

- Routes and states often share the same components. But, stateprovider allows us to have names and nested views
- With ui-router we use ui-view, instead of ng-view

`angular.module('Eggly', [
    'ngAnimate',
    'ui.router',
    'categories',
    'categories.bookmarks'
])
    .config(function ($stateProvider, $urlRouterProvider) {
        $stateProvider
            //abstract state serves as a PLACEHOLDER or NAMESPACE for application states
            .state('eggly', {
                url: '',
                // templateUrl: 'app/categories/categories.tmpl.html',
                abstract: true
            })
        ;

        $urlRouterProvider.otherwise('/');
    })
;`

## Named view - Separating Views
- Defining a top level (ROOT/parent) template and child templates
- So we declare our parent module to have and abstract state (as shown above), and then declare our state in each child module with it's specific view and controller

`
angular.module('categories', [
    'eggly.models.categories'
])
    .config(function ($stateProvider) {
        $stateProvider
            .state('eggly.categories', {
                url: '/',
                views: {
                    //target the ui-view named 'categories' in ROOT state (eggly)
                    'categories@': {
                        controller: 'CategoriesListCtrl as categoriesListCtrl',
                        templateUrl: 'app/categories/categories.tmpl.html'
                    },
                    //target the ui-view named 'bookmarks' in ROOT state (eggly)
                    //to show all bookmarks for all categories
                    'bookmarks@': {
                        controller: 'BookmarksListCtrl as bookmarksListCtrl',
                        templateUrl: 'app/categories/bookmarks/bookmarks.tmpl.html'
                    }
                }
            })
        ;
    })`

- The @ sign mean absolute path

- Then we could just chain a controller as follows.

`
    .controller('CategoriesListCtrl', function CategoriesListCtrl(CategoriesModel) {
        var categoriesListCtrl = this;

        CategoriesModel.getCategories()
            .then(function (result) {
                categoriesListCtrl.categories = result;
            });
    })
;`


## State Params
How to pass information from one state to another
The question is how do we pass the selected 'items' from one state to another to keep track of 

- We can use $stateparams
- We can just define it in the URL

`angular.module('categories.bookmarks', [
    'categories.bookmarks.create',
    'categories.bookmarks.edit',
    'eggly.models.categories',
    'eggly.models.bookmarks'
])
    .config(function ($stateProvider) {
        $stateProvider
            .state('eggly.categories.bookmarks', {
                url: 'categories/:category',
                //target the named 'ui-view' in ROOT (eggly) state named 'bookmarks'
                //to show bookmarks for a specific category
                views: {
                    'bookmarks@': {
                        templateUrl: 'app/categories/bookmarks/bookmarks.tmpl.html',
                        controller: 'BookmarksListCtrl as bookmarksListCtrl'
                    }
                }
            })
        ;
    })`

- Then we can make use that parameter in the controller

`    .controller('BookmarksListCtrl', function ($stateParams, CategoriesModel, BookmarksModel) {
        var bookmarksListCtrl = this;

        CategoriesModel.setCurrentCategory($stateParams.category);

        BookmarksModel.getBookmarks()
            .then(function (bookmarks) {
                bookmarksListCtrl.bookmarks = bookmarks;
            });

        bookmarksListCtrl.getCurrentCategory = CategoriesModel.getCurrentCategory;
        bookmarksListCtrl.getCurrentCategoryName = CategoriesModel.getCurrentCategoryName;
        bookmarksListCtrl.deleteBookmark = BookmarksModel.deleteBookmark;
    })

;`

## Navigating between states with ui-router

- One way to navigate through states is through the URL, the other ways is by calling $state.go('the state we want to go throuhg', {object map of the parameter we want to pass in});

`$state.go('eggly.categories.bookmarks', {categories:categories.name});`
 
- The second way is through `ui-sref` in html 

`<a ui-sref="eggly.categories.bookmarks({category:category.name})">
            {{category.name}}
        </a>`

- Then we would no longer need to use ng-click

## Controller as syntax
-Implicit $scope inheritance can be avoided by us the 'controllers as' syntax  

# Data modeling
- Promoting collection to models by making them available to the rest of the application
- Leverage services

`angular.module('eggly.models.bookmarks', [])
    .service('BookmarksModel', function ($http, $q) {
        var model = this,

        ....`

- Then in the controller, in this case the 'categories.bookmarks' controller, inject the 'BookmarksModel'
- And use getter functions defined in the model services
`

...
    .controller('BookmarksListCtrl', function ($stateParams, CategoriesModel, BookmarksModel) {
        var bookmarksListCtrl = this;

        CategoriesModel.setCurrentCategory($stateParams.category);

        BookmarksModel.getBookmarks()
            .then(function (bookmarks) {
                bookmarksListCtrl.bookmarks = bookmarks;
            });

        bookmarksListCtrl.getCurrentCategory = CategoriesModel.getCurrentCategory;
        bookmarksListCtrl.getCurrentCategoryName = CategoriesModel.getCurrentCategoryName;
        bookmarksListCtrl.deleteBookmark = BookmarksModel.deleteBookmark;
    })

;`

## Using $http to load JSON data
- Uses JSONP
- Create an endpoint map, it looks cleaner

- Our controllers should not know about the implementation details of how our data is being returned, so let's abstract it. Check the `extract` functions and `cacheCategories` which is called by `getCategories`


`
angular.module('eggly.models.categories', [

])
    .service('CategoriesModel', function ($http, $q) {
        var model = this,
            URLS = {
                FETCH: 'data/categories.json'
            },
            categories,
            currentCategory;

        function extract(result) {
            return result.data;
        }

        function cacheCategories(result) {
            categories = extract(result);
            return categories;
        }

        model.getCategories = function() {
            return (categories) ? $q.when(categories) : $http.get(URLS.FETCH).then(cacheCategories);
        };

        model.setCurrentCategory = function(category) {
            return model.getCategoryByName(category).then(function(category) {
                currentCategory = category;
            })
        };

        model.getCurrentCategory = function() {
            return currentCategory;
        };

        model.getCurrentCategoryName = function() {
            return currentCategory ? currentCategory.name : '';
        };

        model.getCategoryByName = function(categoryName) {
            var deferred = $q.defer();

            function findCategory(){
                return _.find(categories, function(c){
                    return c.name == categoryName;
                })
            }

            if(categories) {
                deferred.resolve(findCategory());
            } else {
                model.getCategories()
                    .then(function() {
                        deferred.resolve(findCategory());
                    });
            }

            return deferred.promise;
        };


    })
;
`

## Control your promises with $q

- Check in the above how we manually resolve these promises when, for example we want to cache the query response

- There are situation where you want to do something to that data before passing it to the controller (i.e. return a cache instead of performing a call).
THis is where the $q service comes in.

- $q.when allows you to wrap a promise object around the data 

- $q.defer() - creates a deferred object

## Edit and Create Bookmar States

- Cleaning the the controller(s)
- Essentially we would delegate responsibility to individual modules that handle their own individual template and controllers
-- These are nested, meaning that in a single page, you can have more than one modules.

## Animate state transition with ui-router using ng-animate

- Add angular-animate.js resource to the index.html, and add `ngAnimate` as dependency to the app.js file
- CSS
`
[ui-view].ng-enter, [ui-view].ng-leave {
    position: absolute;
    left: 0;
    right: 0;
    -webkit-transition:all .5s ease-in-out;
    transition:all .5s ease-in-out;
}

[ui-view].ng-enter {
    opacity: 0;
}

[ui-view].ng-enter-active {
    opacity: 1;
}

[ui-view].ng-leave {
    opacity: 1;
}

[ui-view].ng-leave-active {
    opacity: 0;
}
`



