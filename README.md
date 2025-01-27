# Laravel Guidelines at Protone Media

Work in progress!

## Development Guidelines

### Dependencies and styling

* Use the latest version of the [Laravel Framework](https://laravel.com/docs/9.x) and additional packages.
* Use the least amount of additional packages, and avoid solutions that require the installation of additional tools on the server.
* Stay close to first-party Laravel-style conventions and guidelines. Carefully read the framework documentation and navigate its source code and first-party packages for inspiration.

### General

* Everything should work with [Inertia SSR](https://inertiajs.com/server-side-rendering) and [Laravel Octane](https://laravel.com/docs/9.x/octane).
* Use translation strings, don't hard-code strings. This also applies to data like the app name, company name, company address, etc. Things may change over time.
* Prefer storing settings in the database rather than configuration files.
* Avoid events/listeners. As always, it depends on the situation. If we're talking about 5+ actions, this might become cumbersome, and you want to use listeners. But my starting point is to keep it simple within the controller and refactor when it becomes more complex.
* Use [atomic locks](https://laravel.com/docs/9.x/cache#atomic-locks) for actions like account creation, order confirmation, etc.

### Testing

* Write tests for everything. Also, every endpoint should have a [Laravel Dusk](https://laravel.com/docs/9.x/dusk) E2E test, and every feature should have its own test.

### Controllers and Actions

* Keep controllers minimalistic and use actions similar to [Laravel Fortify](https://laravel.com/docs/9.x/fortify). For example, an interface would be named `UpdatesUserProfileInformation`, and its implementation `UpdateUserProfileInformation`.
* Keep an activity log of all non-GET requests.

### Security

* Implement CSP and other security-related headers from the start. Don't think of it as an after sight.

### Database and Eloquent

* Create sensible [Model Factories](https://laravel.com/docs/9.x/database-testing#defining-model-factories) and [Seeders](https://laravel.com/docs/9.x/database-testing#running-seeders). After cloning this repo, there should be a single seeder that you can run to interact with all parts of the app.
* If you need to display a set of database records, always use [pagination](https://laravel.com/docs/9.x/pagination).
* If you need to loop over database records and do some work, always use [chunks](https://laravel.com/docs/9.x/queries#chunking-results) with the `chunk` or `chunkById` method.
* Never leak Eloquent Models into the front-end. Always use [API Resources](https://laravel.com/docs/9.x/eloquent-resources) and don't use `toArray()` on a Model. Also, never directly use a request to save a model (e.g. `Model::create($request->all())`). Always validate and manually specify all fields  (e.g. `Model::create($request->validated())`). This way, we can unguard the models.
* Use the static `query()` method to begin querying an Eloquent Model.
* Use Incrementing IDs as the internal primary key. Use UUIDs for consumer-facing endpoints.
* Always [prevent the lazy loading](https://laravel.com/docs/9.x/eloquent-relationships#preventing-lazy-loading) of relationships.
* Keep an eye on the duration of individual database queries. You may add this snippet, which I found in the [`handleExceedingCumulativeQueryDuration` PR](https://github.com/laravel/framework/pull/42744#issue-1267053567).

```php
if (!app()->isProduction()) {
    DB::listen(function (QueryExecuted $event) {
        if ($event->time > 100) {
            throw new QueryException(
                $event->sql,
                $event->bindings,
                new Exception("Individual database query exceeded 100ms.")
            );
        }
    });
}
```

* For all data, write a mechanism to delete it as well. Make sure files and database records are *deletable* without breaking the application.

### Mailables and Notifications

* Prefer attaching a PDF to a Mailable rather than using more text or data in the mail contents.
* There must be a way for users to resend a Mailable, for example, an order confirmation.

## ESLint

You may use [ESLint](https://eslint.org) to find problems that can be automatically fixed. You'll find a sensible default `.eslintrc.js` file in this repository.

```bash
npm install eslint eslint-plugin-vue --dev
```

NPM script:

```json
"scripts": {
    "eslint": "./node_modules/.bin/eslint resources/js/ --ext .js,.vue --fix"
}
```

## PHP CS Fixer

You may use [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) to fix your code to follow standards. You'll find a sensible default `.php-cs-fixer.php` file in this repository.

```bash
composer require friendsofphp/php-cs-fixer --dev
```

Composer script:

```json
"scripts": {
    "php-cs-fixer": [
        "vendor/bin/php-cs-fixer fix --config=.php-cs-fixer.php --verbose"
    ],
}
```

## Babel

You may use [Babel](https://babeljs.io/docs/en/babel-plugin-syntax-dynamic-import) to support [code splitting with Inertia](https://inertiajs.com/client-side-setup#code-splitting).

```bash
npm install @babel/plugin-syntax-dynamic-import --dev
```

## Laravel Mix

Install [Polyfill extension](https://laravel-mix.com/extensions/polyfill) to include polyfills by using Babel, core-js, and regenerator-runtime.

```
npm install laravel-mix-polyfill --dev
```

Checkout the `webpack.mix.js` example file, which includes fixes for some older browsers.