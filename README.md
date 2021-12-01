# JReviews 4 Release notes

JReviews 4 is currently in Beta which means you shouldn't use it in a production site because the code is still a work in progress. 

There's also a release [CHANGELOG](CHANGELOG.md) where you will find the most important changes between updates.

As mentioned in the [The Next Evolution of JReviews](https://www.jreviews.com/blog/the-next-evolution-of-jreviews) blog post, while JReviews 4 introduces a parallel framework, the codebase for JReviews itself, at this time, and remains for the most part the same as JReviews 3.

If you have made, or plan on making customizations, the code changes will be felt mostly on new developments and any functionality that undergoes a re-write to use the new framework. At this time the MyLists Add-on was re-written almost in its entirety to use the new framework, and there's also a new Events Calendar Add-on built entirely on the new framework that is only available for JReviews 4.

The following notes are for reference purposes for use during the development of JReviews 4 and will be updated as needed.

- [Installation](#installation)
- [Templating](#templating)
	- [Template Overrides](#template-overrides)
	- [Theme Suffixes](#theme-suffixes)
- [Locale](#locale)
- [Hooks](#hooks)
- [Events & Listeners](#events-and-listeners)
	- [Defining Events](#defining-events)
	- [Defining Listeners](#defining-listeners)
	- [Queued Event Listeners](#queued-event-listeners)
	- [Registering Listeners](#registering-listeners)
	- [Dispatching Events](#dispatching-events)
	- [Running a Queue Worker](#running-a-queue-worker)
- [Code Overrides](#code-overrides)

## Installation

If you want to test JReviews 4 Alpha that would be very much appreciated it and you can download it from the client area.

If you are creating a brand new JReviews site, download the package and install as usual per the [documentaiton](https://www.jreviews.com/docs/installation).

If you are upgrading an existing site:

- Deploy a new development site.
- Uninstall the MyLists Add-on if you use it.

	Skipping this step will result in a fatal error after installing JReviews. If you ended up here (with a fatal error), access your server using FTP and find the mylists directory add-on under /components/com_jreviews_addons in Joomla and /wp-content/plugins/jreviews_addons in WP and rename or delete the directory.
	
- Uninstall both S2Framework and JReviews from your site. 
	
	This will just get rid of the files, not the data stored in the database nor media files. For now, given the limited number of changes in JReviews you can continue using your existing JReviews 3 customizations for JReviews and most add-ons (except MyLists).

- Update MyLists, Import and RapidLauncher Add-ons if you already have them installed.
- Install the new and free Events Calendar Add-on and take it for a spin! 
- There's a new forum board for [JReviews 4 general discussion](https://forum.jreviews.com/forum/134-general-discussion-before-stable-release/) during the alpha, beta stages.

## Templating

JReviews 4 implements Laravel's Blade templating engine. You can learn more about Blade directly from the [Laravel Documentation](https://laravel.com/docs/8.x/blade)

JReviews 4 is dropping the multiple theme functionality in favor of a single theme. The original idea for multiple themes was to allow users and 3rd parties to develop themes for JReviews, but this never materialized. 

Whereas in JReviews 3 the themes are located within JReviews in the `/jreviews/views` directory. In JReviews 4, the templates are located within the `/resources/views` and the same applies for Add-ons. 

> :warning: Theme Explorer-like functionality, dark mode and RTL are not available for JReviews 4 at this time.

### Template Overrides

To override a template file in JReviews you just need to place a copy of the template in the overrides directory. For example, to override a MyLists Add-on template file this would be the path:

**Joomla**

`templates/jreviews_overrides/addons/mylists/resources/views/site/yoyo/lists-page.blade.php`

**Joomla**

`jreviews_overrides/addons/mylists/resources/views/site/yoyo/lists-page.blade.php`

### Theme Suffixes

Like with JReviews 3, you can also have different overrides for the same template when applying a theme suffix. You need to append the suffix to the name of the file `lists-page_suffixed.blade.php`  where the suffix is `_suffixed`.

## Locale

There's a new language system in JReviews 4 which uses PHP arrays to define the language keys and strings. The core language files for JReviews and Add-ons can be found in the `resources\lang` directory where each language will have it's own sub-directory. 

The front-end strings are in `site.php`, and admin strings are in `cp.php`. 

To override an existing language string or add missing strings for different languages for JReviews or an Add-on, first create an empty file in overrides for the specific language.

**Joomla**

`/templates/jreviews_overrides/resources/lang/en/site.php`

**WordPress**

`/jreviews_overrides/resources/lang/de/site.php`

The files should return an array with the desired strings:

```php
<?php

return [
	'Search' => 'Suche',
];
```

The files in the `resources/lang` directory are used as fallback for all strings in JReviews and Add-ons. If you'd rather separate your overrides for JReviews and Add-ons into individual directories, JReviews 4 also allows you to do that. For example:

**Joomla**

```
/templates/jreviews_overrides/resources/lang/vendor/jreviews/de/site.php`
/templates/jreviews_overrides/resources/lang/vendor/mylists/de/site.php`
```

**WordPress**

```
/jreviews_overrides/resources/lang/vendor/jreviews/en/site.php`
/jreviews_overrides/resources/lang/vendor/mylists/en/site.php`
```

## Hooks

JReviews 4 implements a new hooks system similar to JReviews 3, but uses different class names. At this time the only hooks using the new system are:

### MyLists

- mylists:list_form_bottom
- mylists:list_form_validation

### EventsCalendar

**Starting with v1.0.4:**

- eventscalendar:status-badge.before-status
- eventscalendar:status-badge.after-status
- eventscalendar:month-view.before-event-title
- eventscalendar:month-view.after-event-title
- eventscalendar:day-view.after-last-field

### Action Hooks

To create an action hook the following syntax is used:

```php
FWDHook::action('my.action', $user);
```

To hook into the above action you would use:

```php
FWDHook::addAction('my.hook', function($user) {
    if ($user->is_awesome) {
         $this->doSomethingAwesome($user);
    }
}, 20, 1);
```

 The third argument is the priority of the hook. The lower the number, the earlier the execution. The fourth parameter specifies the number of arguments your listener accepts.
 
 Within Blade templates it's also possible to hook into an action with a directive:
 
 ```blade
 @action('my.hook', $user)
 ```
 
 ### Filter Hooks
 
To create a filter the following syntax is used:

```php
$value = FWDHook::filter('my.hook', 'awesome');
```

To hook into the above filter you would use:

```php
FWDHook::addFilter('my.hook', function($what) {
    $what = 'not '. $what;
    return $what;
}, 20, 1);
```

Within blade templates you can use the filter directive:

```blade
You are @filter('my.hook', 'awesome')
```

## Events and Listeners

A new events system is implemented in JReviews 4, and it's only available for events dispatched through the new framework. At this time, the only events dispatched with the new implementation are:

- JReviews\Addons\MyLists\App\Events\ListingWasAddedToList::class
- JReviews\Addons\MyLists\App\Events\ListingWasRemovedFromList::class

### Defining Events

An event class holds information about the event and typically doesn't contain logic.

```php
<?php

namespace JReviews\Addons\MyLists\App\Events;

use FWD\Illuminate\Foundation\Events\Dispatchable;
use FWD\Illuminate\Queue\SerializesModels;
use JReviews\Addons\MyLists\App\Models\ListListing;

class ListingWasAddedToList
{
    use Dispatchable;

    public $listListing;

    public function __construct(ListListing $listListing)
    {
        $this->listListing = $listListing;
    }
}
```

### Defining Listeners

Event listeners receive event instances in their handle method.

```php
<?php

namespace JReviewsOverrides\Addons\MyLists\App\Listeners;

use JReviews\Addons\MyLists\App\Events\ListingWasAddedToList;

class DoSomething
{
    public function handle(ListingWasAddedToList $event)
    {
        // Do something with $event->listListing
    }
}
```

### Queued Event Listeners

> :warning: Support for queued listeners in JReviews 4 is still a work in progress given that none of the JReviews 3 core events have been migrated to the new events system. You should continue using the Queue Add-on to process your queues as usual.

Queueing listeners can improve the performance of any given action by performing slower tasks asynchronously, such as sending an email or making an HTTP request.

To specify that a listener should be queued, the listener should implement the `shouldQueue` interface. The connection type also needs to be set to `database` either via the $connection property or programmatically by returning the value from the `viaConnection` method. If you return `sync` instead of `database`, the listener will not be queued and will be processed in the same request.

```php
<?php

namespace JReviewsOverrides\Addons\MyLists\App\Listeners;

use JReviews\Addons\MyLists\App\Events\ListingWasAddedToList;
use FWD\Illuminate\Contracts\Queue\ShouldQueue;

class DoSomething implements ShouldQueue
{
		/**
		* Return 'sync' to execute the listener in the same request
		*/
		public function viaConnection()
		{
			return 'database';
		}

    public function handle(ListingWasAddedToList $event)
    {
        // Do something with $event->listListing
    }
}
```

To process the queue it is necessary to configure a process monitor such as Supervisor to permanently run the queue worker in the background. Refer to the [Running a Queue Worker](#running-a-queue-worker) section below for more information.

### Registering Listeners

To register listeners for existing events create an 'events.php` configuration file in overrides:

| **Joomla** | **WordPress** |
| ---------- | ------------- |
| `/templates/jreviews_overrides/config/events.php` | `/jreviews_overrides/config/events.php` |

The file should return an array where the array keys are the fully qualified class name for the event, and similarly the values for each key are one or more fully qualified class names for listeners. For example:

```php
<?php

return [
    JReviews\Addons\MyLists\App\Events\ListingWasAddedToList::class => [
        JReviewsOverrides\Addons\MyLists\App\Listeners\DoSomething::class
    ],
];
```

The listener class name (and namespace can be anything you want, as long as the class is already (auto)loaded. However, if the namespace begins with `JReviewsOverrides\App` for JReviews core classes and `JReviewsOverrides\Addons` for Add-on classes and follow PSR-4 naming conventions, then the class will be autoloaded for you. In the above example, the file for the DoSomething class should be located at:

`/jreviews_overrides/addons/mylists/app/Listeners/DoSomething.php`

### Dispatching Events

To dispatch an event you just have to statically call the `dispatch` method on the event class which is available via the `Dispatchable` trait.

```php
JReviews\Addons\MyLists\App\Events\ListingWasAddedToList::dispatch($listListing);
```

### Running a Queue Worker

> :warning: Support for queued listeners in JReviews 4 is still a work in progress given that none of the JReviews 3 core events have been migrated to the new events system. You should continue using the Queue Add-on to process your queues as usual.

JReviews 4 includes a new implementation for queued listeners, but since it's not currently in use, if you are using Queue Add-on in JReviews 3, you should continue using it.

In JReviews 4 you would continue to use a process monitor like Supervisor to run the queue worker. In the supervisor configuration you can specify the following command:

**Joomla**

```bash
php /server/path/to/components/com_s2framework/artisan queue:work database 
```

**WordPress**

```bash
php /server/path/to/wp-content/plugins/s2framework/artisan queue:work database 
```

It's also possible to process a single job from the queue programmatically via the `FWDArtisan` facade.

```php
FWDArtisan::call('queue:work database --once');
```

When using named queues, you may also specify a queue name (i.e. `emails`)

```php
FWDArtisan::call('queue:work database --once --queue=emails');
```

## Code Overrides

### PHP Classes

In JReviews 4 you can override individual PHP class methods by extending a base class without the need for overriding the entire file. This approach works for classes that are resolved from the service container because you can specify the original class name as well as the class alias you wish to use for that class. 

To override a class it is necessary to map the original class to the override class in:

`jreviews_overrides/classmap.php`

The file returns an array with a one-to-one relation of key => value, where the key is the name of the original class you want to override, and the value is the name of the new class that extends the original class. Below an example that overrides the User model, and the MyListsPage class for the MyLists Add-on.

```php
<?php

return [
    JReviews\App\Models\User::class => JReviewsOverrides\App\Models\User::class,
    JReviews\Addons\MyLists\App\Http\Site\Yoyo\MyListsPage::class => JReviewsOverrides\Addons\MyLists\App\Http\Site\Yoyo\MyListsPage::class,
];
```

It's important that the overriding class extends the original class. For example:

```php
<?php

namespace JReviewsOverrides\App\Models;

class User extends \JReviews\App\Models\User
{
    // ... 
}
```

The overriding classes will be autoloader as needed as long as the class and file names follow PSR-4 naming conventions and the namespace of overriding classes begins with `JReviewsOverrides`.

#### JReviews

To overrides PHP files in the JReviews' `/app` directory, these need to be placed in `jreviews_overrides/app`

#### Add-ons

To overrides PHP files located in an add-on's `/app` directory, these need to be placed in `jreviews_overrides/addons/ADDON_NAME/app` where ADDON_NAME is replaced with the actual directory name of the add-on. For example:

`jreviews_overrides/addons/mylists/app`
