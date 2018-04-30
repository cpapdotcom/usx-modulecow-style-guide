# ModuleCow Style Guide &mdash; USX Laravel

*The USX approach to ModuleCow's Laravel specific architecture*

##### Other Style Guides

  - [USX JavaScript](https://github.com/cpapdotcom/usx-javascript-style-guide)
  - [USX CSS & Sass](https://github.com/cpapdotcom/usx-css-sass-style-guide)
  - [USX Vue 2](https://github.com/cpapdotcom/usx-vue2-style-guide)

## Table of Contents

  1. [Terminology](#terminology)
  1. [Directory Structure](#directory-structure)
  1. [Classes](#classes)
  1. [Controllers](#controllers)
  1. [Routing](#routing)
  1. [Arrays](#arrays)
  1. [Database Interactions](#database-interactions)

## Terminology
  <a name="terminology--list"></a><a name="1.1"></a>
  - [1.1](#terminology--list) **List**: List of Laravel terminology.

  | Term | Definition |
  | ---- | ---------- |
  | **[Service Providers](https://laravel.com/docs/5.5/providers)**        | Service providers are classes that are instantiated during the application bootstrapping process (which occurs on every HTTP request). These are used to register all the application bindings, event listeners, middleware, routes, and more. Service providers are the central place to configure your application. |
  | **[Route](https://laravel.com/docs/5.5/routing)**                      | Laravel is an actual app, and as such it doesn't simply load static PHP files. Instead, it uses routing to direct the HTTP or API request to the proper Controller and the logic and response are handled in the application logic. Then you simply define endpoints and responses by listing the URI and logic handler in a routes file. |
  | **[Middleware](https://laravel.com/docs/5.5/middleware)**              | Middleware is a powerful method of injecting logic between either an incoming request or outgoing response and the client. Middleware is incredibly useful for authentication, authorization, verifying CSRF tokens, sending global data back to views, and more. Really anything that could stop, change, or log an incoming request before it reaches its Controller or for passing data back to the user on every response. Middleware provide a convenient mechanism for filtering HTTP requests entering your application. |
  | **[Controller](https://laravel.com/docs/5.5/controllers)**             | The "C" in MVC, Controllers group related request handling logic into a single class. Controller should be concise and pass any sophisticated logic to services or application classes more suited to the task. |
  | **[Request](https://laravel.com/docs/5.5/requests)**                   | Laravel's instance of the current incoming HTTP request. |
  | **[View](https://laravel.com/docs/5.5/views)**                         | The "V" in MVC, Views are the presentation logic of your application. Views contain the HTML served by your application and separate your controller / application logic from your presentation logic. Laravel uses Blade templating engine for view rendering. |
  | **[Blade](https://laravel.com/docs/5.5/blade)**                        | Blade is the simple, yet powerful templating engine provided with Laravel. It is similar to Twig except that Blade is an extension of PHP and thus allows you to use native PHP functions within your view layer which is incredibly useful (think ucfirst, DateTime output, strtotime, etc). blade of course has a different syntax than Twig but it follows the same patterns of conditionals and loops to output content. |
  | **[Log](https://laravel.com/docs/5.5/errors#logging)**                 | Log is Laravel's internal abstract layer on MonoLog, very similar to our shared Logger class in USX shared classes. Within ModuleCow you should no longer use the Logger class and instead use the native Laravel Log class. |
  | **[Session](https://laravel.com/docs/5.5/session)**                    | Laravel's internal session handling instance. |
  | **[Artisan](https://laravel.com/docs/5.5/artisan)** (Console)          | Artisan is the command-line interface included with Laravel. Custom Laravel commands may be easily added and use the convenient Symfony Console classes which make writing CLI commands very easy. These can also easily be used by the Task Scheduler (more on that below) to run commands as crons. |
  | **[Broadcasting](https://laravel.com/docs/5.5/broadcasting)**          | Broadcasting is Laravel's internal system for Websocket integration. ModuleCow currently doesn't support WebSockets but hopefully it will soon, so this is a good term to be familiar with. |
  | **[Cache](https://laravel.com/docs/5.5/cache)**                        | Laravel's internal caching class. Laravel provides an expressive, unified API for various caching backends. |
  | **[Collections](https://laravel.com/docs/5.5/collections)**            | The Collection class provides a fluent, convenient wrapper for working with arrays of data. This is really a powerful helper when dealing with array data and makes it easy to modify, pluck, convert, validate, find, and more. All methods are available at https://laravel.com/docs/5.5/collections#available-methods. Also, a form of the Collections class is also used interacting with Eloquent Model data returned from the database. More on that below. |
  | **[Events](https://laravel.com/docs/5.5/events)**                      | Laravel's events provide a simple observer implementation, allowing you to subscribe and listen for various events that occur in your application. You may set up Event broadcasters and listeners to perform certain logic when a defined application event occurs. For example we use listeners on login success and failure to populate the `LoginLogTbl`. |
  | **[Storage](https://laravel.com/docs/5.5/filesystem)** (File Storage)  | Laravel provides a powerful filesystem abstraction thanks to the wonderful Flysystem PHP package. This makes it much easier to work with, fetch, and store file data. |
  | **[Helpers](https://laravel.com/docs/5.5/helpers)** (Helper Functions) | Laravel provides a load of global helper functions that may be used anywhere within the application! You should really take a moment to skim through them so you are not caught off guard when they are used within the logic with no definition in sight: https://laravel.com/docs/5.5/helpers. |
  | **[Mail](https://laravel.com/docs/5.5/mail)**                          | Laravel Email: Laravel provides a clean, simple API over the popular SwiftMailer library. This makes it much easier to work with sending emails from servers and even utilizes Blade templates to keep application logic separate from the presentation logic. |
  | **[Queues](https://laravel.com/docs/5.5/queues)**                      | Currently ModuleCow doesn't have a queue handler, but hopefully it will soon. Laravel makes it easy to create and use queueable tasks. The Mail class listed above is queueable out of the box. |
  | **[Task Scheduling](https://laravel.com/docs/5.5/scheduling)** (Tasks) | In the past, you may have generated a Cron entry for each task you needed to schedule on your server. However, this can quickly become a pain, because your task schedule is no longer in source control and you must SSH into your server to add additional Cron entries. Laravel's command scheduler allows you to fluently and expressively define your command schedule within Laravel itself. When using the scheduler, only a single Cron entry is needed on your server.. |
  | **Model**                                                              | The "M" in MVC. Models are incredibly powerful abstraction layers for our database tables. What this means is we can define a single source of truth for data and relationships about a table and utilize the model to interact with the table instead of SQL directly. This provides the benefit of declaring relationships with other models (tables) to easily access joined data. It also takes care of the table name (meaning if the table name is changed you only update it once in the model and all model calls will still work fine), timestamps (CreatedAt, UpdatedAt, and DeletedAt for soft deletes), soft deletion, [casting](https://laravel.com/docs/5.5/eloquent-mutators#attribute-casting), automatic slugging, custom faux fields, and so much more. |
  | **[Eloquent](https://laravel.com/docs/5.5/eloquent)**                  | Eloquent is Laravel's ORM system for interacting with Models. The Eloquent ORM included with Laravel provides a beautiful, simple ActiveRecord implementation for working with your database. Each database table has a corresponding "Model" which is used to interact with that table. Models allow you to query for data in your tables, as well as insert new records into the table. |
  | **[Query Builder](https://laravel.com/docs/5.5/queries)**              | Laravel's Query Builder should only be used when queries are too complex for Eloquent. Currently the initial ModuleCow uses only Eloquent. Query Builder should really be a last resort. This is because it bypases everything defined in the Model. We lose the DRYness of relationships and tables naming, and all the defined Model logic, relationships, and helpers. Your returned collection from the Query Builder has only a fraction of the methods available through an Eloquent query, and lastly, the speed improvements are very minor (look for better methods of speeding up your queries when possible such as indexing, additional/better query filters, or paring down your dataset). That all being said, sometimes Query Builder is the better/only solution. Laravel's database query builder provides a convenient, fluent interface to creating and running database queries. |


## Directory Structure
  <a name="directory-structure--application"></a><a name="2.1"></a>
  - [2.1](#directory-structure--application) **Application**: ModuleCow's boilerplate application root directory structure.

  ```
  ├─ bootstrap     // The Laravel application bootstrap directory. Only app.php should ever be modified in this directory.
    ├─ cache       // The application bootstrap cache directory. This should never be modified.
  ├─ config        // The application configuration files. These are often populated from environment variables.
  ├─ database      // The application database migration and seed directory. We currently don't use this feature of Laravel.
    ├─ factories   // Factories are used as Model seeding prototypes that can be used to populate large sets of data.
    ├─ migrations  // Stores the database migrations.
    ├─ seeds       // Seed dummy data into Models for sandbox and testing
  ├─ modules       // Contains the actual application modules. See the next directory structure section below.
  ├─ public        // This is the public directory. This should remain sparse. It's mostly populated by assets builds.
  ├─ resources     // Contains the ModuleCow global resources used accross all modules.
    ├─ assets      // Front-end assets such as JS, Vue, and SASS files. Currently only contains global JS app files.
    ├─ lang        // Language variations for Blade output (not currently used)
    ├─ views       // Global Blade templates and Blade error pages
  ├─ storage       // Contains files generated by the Laravel framework. These should never be modified manually.
  ├─ tests         // Contains automated tests.
  ├─ vendor        // Vendor directory created by Composer PHP package manager. This should never be modified.
  ├─ node_modules  // Vendor directory created by Yarn JS package manager. This should never be modified.
  ```
  > **Note:** The above *Application* directory structure is a concise and ModuleCow specific version of Laravel's [Root Directory Structure](https://laravel.com/docs/5.5/structure#the-root-directory) documentation. It's highly recommended you read through Laravel's documentation as well.

  <a name="directory-structure--modules"></a><a name="2.2"></a>
  - [2.2](#directory-structure--modules) **Modules**: ModuleCow's boilerplate `modules` directory structure.

  ```
  ├─ modules            // The root module directory that contains each app module
    ├─ MODULE_NAME      // The name of the ModuleCow module
      ├─ Config         // Contains the module config file created by laravel-modules plugin
      ├─ Events         // Contains the event broadcaster classes for the module
      ├─ Exceptions     // Contains custom Exception handlers for the module
      ├─ Http           // HTTP request handling classes for the module
        ├─ Controllers  // Controller classes
        ├─ Middleware   // Middleware classes
        ├─ Routes       // Contains the route definition files for web and API routes
      ├─ Jobs           // Contains the queueable jobs classes (logic for a queued process)
      ├─ Listeners      // Contains the event listener classes for the module
      ├─ Mail           // Email handler classes for the module
      ├─ Models         // Model classes (core Framework module only)
      ├─ Observers      // Model event observer classes (core Framework module only)
      ├─ Providers      // Service provider classes
      ├─ Resources      // Contains module resources for view layer
        ├─ assets       // Front-end assets such as JavaScript, Vue, and SASS files
        ├─ lang         // Language variations for Blade output (not currently used)
        ├─ views        // Blade templates for the module
      ├─ Scopes         // Model scope extension classes (core Framework module only)
      ├─ Services       // Service classes for extended logic and abstraction within the module
      ├─ Tests          // Test classes
      ├─ Traits         // Reusable trait classes for the module
  ```
  > **Note:** The above *Modules* directory structure is a concise and Module specific version of Laravel's [App Directory Structure](https://laravel.com/docs/5.5/structure#the-app-directory) documentation. It's highly recommended you read through Laravel's documentation as well.


## Classes
  <a name="classes--constructor"></a><a name="3.1"></a>
  - [3.1](#classes--constructor) **No Empty Constructor**: All class constructors must contain logic or not be defined.
    > Why? Constructor definitions are not required when no constructor logic is present. Not defining an unnecessary constructor keeps class code cleaner and more concise.

## Controllers
  <a name="controllers--crud-methods"></a><a name="4.1"></a>
  - [4.1](#controllers--crud-methods) **CRUD Methods**: Controllers may only have some or none of the typical RESTful methods, but when one does it should follow these CRUD naming conventions.

  | Action | Controller Method | Request Method | Description |
  | ------ | ----------------- | -------------- | ----------- |
  | List All      | `index()`   | `GET`            | List all entities |
  | Show One      | `show()`    | `GET`            | Show a single entity |
  | Creation Form | `create()`  | `GET`            | Display form for creating a new entity |
  | Save New      | `store()`   | `POST`           | Save/store a new entity in the database |
  | Edit Form     | `edit()`    | `GET`            | Display form for editing an already existing entity |
  | Update        | `update()`  | `PUT` or `PATCH` | Save/update the edited entity |
  | Delete        | `destroy()` | `DELETE`         | Delete or soft delete the entity |

  > A `PUT` request is idempotent and when using `PUT`, it is assumed that you are sending the complete entity, and that complete entity *replaces* any existing entity in full. A `PATCH` request will only update the entity fields passed with the request. Usually a `PUT` is acceptable for complete non-AJAX entity update forms and `PATCH` is used when making an AJAX update that only changes one or several of the entities values on update.

## Routing
  <a name="routing--module-prefix"></a><a name="5.1"></a>
  - [5.1](#routing--module-prefix) **Route Module Prefixing**: *All* routes must begin with the module name prefix.
    > Why? Defines the module in the path for clarification and the dynamic navigation menus.

    **Bad**
    ```php
    Route::get('/employees', 'EmployeeController@index');
    ```

    **Good**
    ```php
    Route::get('/meta/employees', 'EmployeeController@index');
    ```

  <a name="routing--slugs"></a><a name="5.2"></a>
  - [5.2](#routing--slugs) **Dashed Slug Format**: Defined route paths should *always* be in standard dashed slug format.
    > Why? It's the web standard for URI paths.

    **Bad**
    ```php
    Route::get('/route/example/fooBar', 'FooController@bar');
    ```

    **Good**
    ```php
    Route::get('/route/example/foo-bar', 'FooController@bar');
    ```

  <a name="routing--naming"></a><a name="5.3"></a>
  - [5.3](#routing--naming) **Route Naming**: All GET routes should have a route name when defining a route in ModuleCow.
    > Why? Route names are used by our navigation menus and allow for easily recognizable route paths in the codebase.

    **Bad**
    ```php
    Route::get('/route/example/foo-bar', 'FooController@bar');
    ```

    **Good**
    ```php
    Route::get('/route/example/foo-bar', 'FooController@bar')->name('route.example.foo-bar');
    ```

  <a name="routing--naming-convention"></a><a name="5.4"></a>
  - [5.4](#routing--naming-convention) **Route Naming Convention**: Route names should be dot (.) separated with parent path info defined first.
    > Why? Route names are used by our navigation menus and properly nested names are what allow for the navigation menus to display active page highlighting.

    **Bad**
    ```php
    Route::get('/employees', 'EmployeeController@index')->name('employeesIndex');
    Route::get('/employee/create', 'EmployeeController@create')->name('create.employees');
    ```

    **Good**
    ```php
    Route::get('/employees', 'EmployeeController@index')->name('employees');
    Route::get('/employee/create', 'EmployeeController@create')->name('employees.create');
    ```

  <a name="routing--parameters"></a><a name="5.5"></a>
  - [5.5](#routing--parameters) **Route Parameters**: Route parameters should be used in place of querystring GET parameters for common, variable URI data.
    > Why? Route parameters are cleaner, self documenting in route definitions, core to Laravel and the Laravel Route class, and allow for dynamic population within navigation.

    **Bad**

    Route:
    ```php
    Route::get('/order', 'OrderController@show')->name('order.show');
    ```

    Controller:
    ```php
    public function show (Request $request) {
      $orderID = $request->query('orderID');

      $order = Order::findOrFail($orderID);

      ...
    }
    ```

    **Good**

    Route:
    ```php
    Route::get('/order/{id}', 'OrderController@show')->name('order.show');
    ```

    Controller:
    ```php
    public function show (string $id) {
      $order = Order::findOrFail($id);

      ...
    }
    ```

  <a name="routing--ajax"></a><a name="5.6"></a>
  - [5.6](#routing--ajax) **AJAX Routing**: AJAX routes must contain `/ajax/` within the path after the defining parent path prefix.
    > Why? Helps to easily identify that a defined route is for AJAX (XHR) requests and adds consistency.

    **Bad**
    ```php
    Route::patch('/meta/access-control/role/assign-pages', 'RoleController@assignPages');
    ```

    **Good**
    ```php
    Route::patch('/meta/access-control/ajax/role/assign-pages', 'RoleController@assignPages');
    ```

## Arrays
  <a name="arrays--definition"></a><a name="6.1"></a>
  - [6.1](#arrays--definition) **Defining Arrays**: New arrays should be defined using the bracket syntax.
    > Why? Same syntax across JS and PHP to keep consistency within the codebase. It's faster than calling the `array()` function (micro-optimization). It's cleaner and more concise leading to less bloat within the codebase.

    **Bad**
    ```php
    $fooBars = array();

    $foos = array('bar', 'faz', 'baz');
    ```

    **Good**
    ```php
    $fooBars = [];

    $foos = ['bar', 'faz', 'baz'];
    ```

  <a name="arrays--pushing"></a><a name="6.2"></a>
  - [6.2](#arrays--pushing) **Array Data Pushing**: Use the "empty brackets" postfix syntax instead of `array_push()` whenever possible.
    > Why? It's faster than calling the `array_push()` function, especially in loops. It's cleaner and more concise leading to less bloat within the codebase.

    **Bad**
    ```php
    $fooBars = ['foo'];

    array_push($fooBars, 'bar');

    for ($i = 0; $length = 3, $i < $length, $i++) {
      array_push($fooBars, $i);
    }
    ```

    **Good**
    ```php
    $fooBars = [];

    $fooBars[] = 'bar';

    for ($i = 0; $length = 3, $i < $length, $i++) {
      $fooBars[] = $i;
    }

    // When inserting more than one new element it's acceptable to use `array_push()`
    array_push($fooBars, 'faz', 'baz');
    ```

## Database Interactions
  <a name="data--eloquent"></a><a name="7.1"></a>
  - [7.1](#data--eloquent) **Use Eloquent**: Whenever possible [Models](https://laravel.com/docs/5.5/eloquent#defining-models) and [Eloquent](https://laravel.com/docs/5.5/eloquent) should be used to create, read, update, and delete database table data.
    > Why? Models provide an incredibly powerful abstraction layers for our database tables. What this means is we can define a single source of truth for data and relationships about a table and utilize the model to interact with the table instead of SQL directly. This provides the benefit of declaring relationships with other models (tables) to easily access joined data. It also takes care of defining the table name only once, timestamps (CreatedAt, UpdatedAt, and DeletedAt), soft deletion, [casting](https://laravel.com/docs/5.5/eloquent-mutators#attribute-casting), automatic slugging, custom faux fields, and so much more.

    **Bad**
    ```php
    DB::table('UserTbl')->where('userid', 1)->findOrFail();

    // Join data
    DB::table('CustomerTbl')->leftJoin('OrderTbl', 'OrderTbl.CID', '=', 'CustomerTbl.CID')->get();
    ```

    **Good**
    ```php
    User::findOrFail(1);

    // Join data
    Customer::with('order')->get();
    // or with lazy loading
    $customer = Customer::findOrFail(1);

    $orders = $customer->orders;
    ```

  <a name="data--join-condition"></a><a name="7.2"></a>
  - [7.2](#data--join-condition) **Join Condition Order**: When the [Query Builder](https://laravel.com/docs/5.5/queries) must be used with joins the joining table should be listed first in the `ON` join conditional.
    > Why? Provides consistency.

    **Bad**
    ```php
    DB::table('CustomerTbl')->leftJoin('OrderTbl', 'CustomerTbl.CID', '=', 'OrderTbl.CID')->get();
    ```

    **Good**
    ```php
    DB::table('CustomerTbl')->leftJoin('OrderTbl', 'OrderTbl.CID', '=', 'CustomerTbl.CID')->get();
    ```

  <a name="data--specify-table"></a><a name="7.3"></a>
  - [7.3](#data--specify-table) **Specify Table with Complete Table Name**: When it is required to specify a table with a field in the [Query Builder](https://laravel.com/docs/5.5/queries) you should use the full table name, not an obscure abbreviated alias.
    > Why? Provides clarity, easy to understand, semantic, and self documenting.

    **Bad**
    ```php
    DB::table('CustomerTbl AS ct')
        ->leftJoin('OrderTbl AS ot', 'ot.CID', '=', 'ct.CID')->get()
        ->leftJoin('CustomerPhoneTbl AS cpt', function ($join) {
            $join->on('cpt.CID', '=', 'ct.CID');
            $join->on('cpt.SeePhoneFlag', '=', DB::raw(1));
        });
    ```

    **Good**
    ```php
    DB::table('CustomerTbl')
        ->leftJoin('OrderTbl', 'OrderTbl.CID', '=', 'CustomerTbl.CID')->get()
        ->leftJoin('CustomerPhoneTbl', function ($join) {
            $join->on('CustomerPhoneTbl.CID', '=', 'CustomerTbl.CID');
            $join->on('CustomerPhoneTbl.SeePhoneFlag', '=', DB::raw(1));
        });
    ```
