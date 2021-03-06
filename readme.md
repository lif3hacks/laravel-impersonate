# Laravel Impersonate

[![Build Status](https://travis-ci.org/404labfr/laravel-impersonate.svg?branch=master)](https://travis-ci.org/404labfr/laravel-impersonate) [![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/404labfr/laravel-impersonate/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/404labfr/laravel-impersonate/?branch=master)

**Laravel Impersonate** makes it easy to **authenticate as your users**. Add a simple **trait** to your **user model** and impersonate as one of your users in one click.
 
- [Requirements](#requirements)
- [Installation](#installation)
- [Simple usage](#simple-usage)
    - [Using the built-in controller](#using-the-built-in-controller)
- [Advanced Usage](#advanced-usage)
    - [Defining impersonation authorization](#defining-impersonation-authorization)
    - [Using your own strategy](#using-your-own-strategy)
    - [Middleware](#middleware)
    - [Events](#events)
- [Configuration](#configuration)
- [Blade](#blade)
- [Tests](#tests)
- [Contributors](#contributors)


## Requirements

- Laravel >= 5.4
- PHP >= 5.6

## Installation

- Require it with Composer:
```bash
composer require lab404/laravel-impersonate
```

- Add the service provider at the end of your `config/app.php`:
```php
'providers' => [
    // ...
    Lab404\Impersonate\ImpersonateServiceProvider::class,
],
```

- Add the trait `Lab404\Impersonate\Models\Impersonate` to your **User** model.

## Simple usage

Impersonate an user:
```php
Auth::user()->impersonate($other_user);
// You're now logged as the $other_user
```

Leave impersonation:
```php
Auth::user()->leaveImpersonation();
// You're now logged as your original user.
```

### Using the built-in controller

In your routes file you must call the `impersonate` route macro. 
```php
Route::impersonate();
```

```php
// Where $id is the ID of the user you want impersonate
route('impersonate', $id)

// Generate an URL to leave current impersonation
route('impersonate.leave')
```

## Advanced Usage

### Defining impersonation authorization

By default all users can **impersonate** an user.  
You need to add the method `canImpersonate()` to your user model:

```php
    /**
     * @return bool
     */
    public function canImpersonate()
    {
        // For example
        return $this->is_admin == 1;
    }
```

By default all users can **be impersonated**.  
You need to add the method `canBeImpersonated()` to your user model to extend this behavior:

```php
    /**
     * @return bool
     */
    public function canBeImpersonated()
    {
        // For example
        return $this->can_be_impersonate == 1;
    }
```

### Using your own strategy

- Getting the manager:
```php
// With the app helper
app('impersonate')
// Dependency Injection
public function impersonate(ImpersonateManager $manager, $user_id) { /* ... */ }
```

- Working with the manager:
```php
$manager = app('impersonate');

// Find an user by its ID
$manager->findUserById($id);

// TRUE if your are impersonating an user.
$manager->isImpersonating();

// Impersonate an user. Pass the original user and the user you want to impersonate
$manager->take($from, $to);

// Leave current impersonation
$manager->leave();

// Get the impersonator ID
$manager->getImpersonatorId();
```

### Middleware

**Protect From Impersonation**

You can use the middleware `impersonate.protect` to protect your routes against user impersonation.  
This middleware can be useful when you want to protect specific pages like users subscriptions, users credit cards, ... 

```php
Router::get('/my-credit-card', function() {
    echo "Can't be accessed by an impersonator";
})->middleware('impersonate.protect');
```

### Events

There are two events available that can be used to improve your workflow:
- `TakeImpersonation` is fired when an impersonation is taken.
- `LeaveImpersonation` is fired when an impersonation is leaved.

Each events returns two properties `$event->impersonator` and `$event->impersonated` containing User model isntance.

## Configuration

The package comes with a configuration file.  

Publish it with the following command:
```bash
php artisan vendor:publish --tag=impersonate
```

Available options:
```php
    // The session key used to store the original user id.
    'session_key' => 'impersonated_by',
    // The URI to redirect after taking an impersonation.
    // Only used in the built-in controller.
    'take_redirect_to' => '/',
    // The URI to redirect after leaving an impersonation.
    // Only used in the built-in controller.
    'leave_redirect_to' => '/'
```
## Blade

There are three Blade directives available.

### When the user can impersonate

```blade
@canImpersonate
    <a href="{{ route('impersonate', $user->id) }}">Impersonate this user</a>
@endCanImpersonate
```

### When the user can be impersonated

This comes in handy when you have a user list and want to show an "Impersonate" button next to all the users.
But you don\'t want that button next to the current authenticated user neither to that users which should not be able to impersonated according your implementation of `canBeImpersonated()` . 

```blade
@canBeImpersonated($user)
    <a href="{{ route('impersonate', $user->id) }}">Impersonate this user</a>
@endBeImpersonated
```

### When the user is impersonated

```blade
@impersonating
    <a href="{{ route('impersonate.leave') }}">Leave impersonation</a>
@endImpersonating
```

## Tests

```bash
vendor/bin/phpunit
```

## Contributors

- [MarceauKa](https://github.com/MarceauKa)
- [tghpow](https://github.com/tghpow)
- and all others [contributors](https://github.com/404labfr/laravel-impersonate/graphs/contributors)

## Licence

MIT
