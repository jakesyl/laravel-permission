# Associate users with permissions and roles

This package allows you to manage user permissions and roles in a database, this particular modded version allows to assign permissions and roles scoped to a resource, if needed.

It's based on [Spatie](https://github.com/spatie/laravel-permission) package and that is why the namespace has his name and I kept all the referenced to their site: I just modded the package, they are the one to thank for the initial work.

Once installed you can do stuff like this:

```php
// Adding permissions to a user
$user->givePermissionTo('edit-articles');

// Adding permissions via a role
$user->assignRole('writer');

$role->givePermissionTo('edit-articles');
```

If you're using multiple guards we've got you covered as well. Every guard will have its own set of permissions and roles that can be assigned to the guard's users. Read about it in the [using multiple guards](#using-multiple-guards) section of the readme.

Because all permissions will be registered on [Laravel's gate](https://laravel.com/docs/5.4/authorization), you can test if a user has a permission with Laravel's default `can` function:

```php
$user->can('edit-articles');
```

Spatie is a webdesign agency in Antwerp, Belgium. You'll find an overview of all 
their open source projects [on our website](https://spatie.be/opensource).


I work at Dreamonkey Srl, a startup in Scandiano, Italy, as backend and Android developer. You can find a description of who we are [on our website](https://www.dreamonkey.com/) (still no english translation, though).

## Postcardware

You're free to use this package (it's [MIT-licensed](LICENSE.md)), but if it makes it to your production environment Spatie highly appreciate you sending them a postcard from your hometown, mentioning which of their package(s) you are using.

Their address is: Spatie, Samberstraat 69D, 2060 Antwerp, Belgium.

All received postcards are published [on their website](https://spatie.be/en/opensource/postcards).

If you also wanna send a postcard to us, we appreciate it too.

Our address is: Dreamonkey Srl, Via Mazzacurati 3/B, Scandiano 42019, RE, Italy.

## Installation

This package can be used in Laravel 5.4 or higher. If you are using an older version of Laravel, take a look at [the v1 branch of this package](https://github.com/spatie/laravel-permission/tree/v1).

You can install the package via composer.
Being a modded version, it's not present on packagist for now; for this reason you have to manually add the repository to the composer.json file and then add the require line.

``` bash
    ...
    "repositories": [
        ...
        {
            "type": "vcs",
            "url": "https://github.com/IlCallo/laravel-permission"
        }
    ],
    "require": {
        ...
        "spatie/laravel-permission": "^2.2.0"
        ...
    }
```

Now add the service provider in `config/app.php` file:

```php
'providers' => [
    // ...
    Spatie\Permission\PermissionServiceProvider::class,
];
```

You can publish the migration with:

```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="migrations"
```

After the migration has been published you can create the role- and permission-tables by
running the migrations:

```bash
php artisan migrate
```

You can publish the config file with:

```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="config"
```

This is the contents of the published `config/permission.php` config file:

```php
return [

    'models' => [

        /*
         * When using the "HasRoles" trait from this package, we need to know which
         * Eloquent model should be used to retrieve your permissions. Of course, it
         * is often just the "Permission" model but you may use whatever you like.
         *
         * The model you want to use as a Permission model needs to implement the
         * `Spatie\Permission\Contracts\Permission` contract.
         */

        'permission' => Spatie\Permission\Models\Permission::class,

        /*
         * When using the "HasRoles" trait from this package, we need to know which
         * Eloquent model should be used to retrieve your roles. Of course, it
         * is often just the "Role" model but you may use whatever you like.
         *
         * The model you want to use as a Role model needs to implement the
         * `Spatie\Permission\Contracts\Role` contract.
         */

        'role' => Spatie\Permission\Models\Role::class,

    ],

    'table_names' => [

        /*
         * When using the "HasRoles" trait from this package, we need to know which
         * table should be used to retrieve your roles. We have chosen a basic
         * default value but you may easily change it to any table you like.
         */

        'roles' => 'roles',

        /*
         * When using the "HasRoles" trait from this package, we need to know which
         * table should be used to retrieve your permissions. We have chosen a basic
         * default value but you may easily change it to any table you like.
         */

        'permissions' => 'permissions',

        /*
         * When using the "HasRoles" trait from this package, we need to know which
         * table should be used to retrieve your models permissions. We have chosen a
         * basic default value but you may easily change it to any table you like.
         */

        'model_has_permissions' => 'model_has_permissions',

        /*
         * When using the "HasRoles" trait from this package, we need to know which
         * table should be used to retrieve your models roles. We have chosen a
         * basic default value but you may easily change it to any table you like.
         */

        'model_has_roles' => 'model_has_roles',

        /*
         * When using the "HasRoles" trait from this package, we need to know which
         * table should be used to retrieve your roles permissions. We have chosen a
         * basic default value but you may easily change it to any table you like.
         */

        'role_has_permissions' => 'role_has_permissions',
    ],

    /*
     * By default all permissions will be cached for 24 hours unless a permission or
     * role is updated. Then the cache will be flushed immediately.
     */
     
    'cache_expiration_time' => 60 * 24,

    /*
     * By default we'll make an entry in the application log when the permissions
     * could not be loaded. Normally this only occurs while installing the packages.
     *
     * If for some reason you want to disable that logging, set this value to false.
     */

    'log_registration_exception' => true,
];
```

## Usage

First add the `Spatie\Permission\Traits\HasRoles` trait to your User model(s):

```php
use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;
    
    // ...
}
```

This package allows for users to be associated with permissions and roles. Every role is associated with multiple permissions.
A `Role` and a `Permission` are regular Eloquent models. They have a name and can be created like this:

```php
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;

$role = Role::create(['name' => 'writer']);
$permission = Permission::create(['name' => 'edit-articles']);
```

If you're using multiple guards the `guard_name` attribute needs to be set as well. Read about it in the [using multiple guards](#using-multiple-guards) section of the readme.

The `HasRoles` adds Eloquent relationships to your models, which can be accessed directly or used as a base query:

```php
$permissions = $user->permissions;
$roles = $user->roles()->pluck('name'); // Returns a collection
```

The `HasRoles` also adds a scope to your models to scope the query to certain roles:

```php
$users = User::role('writer')->get(); // Only returns users with the role 'writer'
```

The scope can accept a string, a `Spatie\Permission\Models\Role` object or an `\Illuminate\Support\Collection` object.

### Using permissions

A permission can be given to any user with the `HasRoles` trait:

```php
$user->givePermissionTo('edit-articles');

// You can also give multiple permission at once using an array
$user->givePermissionTo(['edit-articles', 'delete-articles']);
```

A permission can be revoked from a user:

```php
$user->revokePermissionTo('edit-articles');
```

You can test if a user has a permission:

```php
$user->hasPermissionTo('edit-articles');
```

Saved permissions will be registered with the `Illuminate\Auth\Access\Gate` class for the default guard. So you can
test if a user has a permission with Laravel's default `can` function:

```php
$user->can('edit-articles');
```

### Using permissions and roles

A role can be assigned to any user with the `HasRoles` trait:

```php
$user->assignRole('writer');

// You can also assign multiple roles at once
$user->assignRole('writer', 'admin');
$user->assignRole(['writer', 'admin']);
```

A role can be removed from a user:

```php
$user->removeRole('writer');
```

Roles can also be synced:

```php
// All current roles will be removed from the user and replace by the array given
$user->syncRoles(['writer', 'admin']);
```

You can determine if a user has a certain role:

```php
$user->hasRole('writer');
```

You can also determine if a user has any of a given list of roles:

```php
$user->hasAnyRole(Role::all());
```

You can also determine if a user has all of a given list of roles:

```php
$user->hasAllRoles(Role::all());
```

The `assignRole`, `hasRole`, `hasAnyRole`, `hasAllRoles`  and `removeRole` functions can accept a
 string, a `Spatie\Permission\Models\Role` object or an `\Illuminate\Support\Collection` object.

A permission can be given to a role:

```php
$role->givePermissionTo('edit-articles');
```

You can determine if a role has a certain permission:

```php
$role->hasPermissionTo('edit-articles');
```

A permission can be revoked from a role:

```php
$role->revokePermissionTo('edit-articles');
```

The `givePermissionTo` and `revokePermissionTo` functions can accept a 
string or a `Spatie\Permission\Models\Permission` object.

Saved permission and roles are also registered with the `Illuminate\Auth\Access\Gate` class.

```php
$user->can('edit-articles');
```

All permissions of roles that user is assigned to are inherited to the 
user automatically. In addition to these permissions particular permission can be assigned to the user too. For instance: 

```php
$role->givePermissionTo('edit-articles');

$user->assignRole('writer');

$user->givePermissionTo('delete-articles');
```

In above example a role is given permission to edit articles and this role is assigned to a user. Now user can edit articles and additionally delete articles. The permission of 'delete articles' is his direct permission because it is assigned directly to him. When we call `$user->hasDirectPermission('delete-articles')` it returns `true` and `false` for `$user->hasDirectPermission('edit-articles')`. 

This method is useful if one has a form for setting permissions for roles and users in his application and want to restrict to change inherited permissions of roles of user, i.e. allowing to change only direct permissions of user.

You can list all of theses permissions:

```php
// Direct permissions
$user->getDirectPermissions() // Or $user->permissions;

// Permissions inherited from user's roles
$user->getPermissionsViaRoles();

// All permissions which apply on the user
$user->getAllPermissions();
```

All theses responses are collections of `Spatie\Permission\Models\Permission` objects.

If we follow the previous example, the first response will be a collection with the 'delete article' permission, the 
second will be a collection with the 'edit article' permission and the third will contain both.

### Using permissions and roles with scopes

If a scope is not explicitly specified, all methods will operate on a not scoped level, which means that a user's roles
and permissions are valid per user.

To grant permissions to a user related to a resource (eg. ability to edit all articles produced by a single department)
it's possible to "scope" the permissions and roles to a particular model, which must implement the `Restrictable` 
interface. A default implementation based on the Eloquent `Model` class can be added to your model(s) with 
the `RestrictableTrait`.

To scope permissions and roles, a `Restrictable` instance can be provided as second parameter to all the methods
which grant, revoke, sync or get permissions or roles.

```php
$role->givePermissionTo('edit-articles');

$user->assignRole('writer', $department);

$user->givePermissionTo('delete-articles', $department);

$user->syncRoles(['writer', 'admin'], $department);

// Scoped direct permissions
$user->getDirectPermissions($department)

// Permissions inherited from user's scoped roles
$user->getPermissionsViaRoles($department);

// All permissions which apply on the user for the given department
$user->getAllPermissions($department);

$user->can('edit-articles', $departement);
```

Do note that Roles and Permissions can be scoped, but scoped permissions assigned to a role are not currently supported.

### Using Blade directives
This package also adds Blade directives to verify whether the currently logged in user has all or any of a given list of
roles. Optionally you can pass in the `guard` that the check will be performed on as a second argument.

```php
@role('writer')
    I'm a writer!
@else
    I'm not a writer...
@endrole
```

```php
@hasrole('writer')
    I'm a writer!
@else
    I'm not a writer...
@endhasrole
```

```php
@hasanyrole(Role::all())
    I have one or more of these roles!
@else
    I have none of these roles...
@endhasanyrole
```

```php
@hasallroles(Role::all())
    I have all of these roles!
@else
    I don't have all of these roles...
@endhasallroles
```

You can use Laravel's native `@can` directive to check if a user has a certain permission.

## Using multiple guards

When using the default Laravel auth configuration all of the above methods will work out of the box, no extra configuration required.

However when using multiple guards they will act like namespaces for your permissions and roles. Meaning every guard has its own set of permissions and roles that can be assigned to their user model.

### Using permissions and roles with multiple guards

By default the default guard (`auth.default.guard`) will be used as the guard for new permissions and roles. When creating permissions and roles for specific guards you'll have to specify their `guard_name` on the model:

```php
// Create a superadmin role for the admin users
$role = Role::create(['guard_name' => 'admin', 'name' => 'superadmin']);

// Define a `create posts` permission for the admin users beloninging to the admin guard
$permission = Permission::create(['guard_name' => 'admin', 'name' => 'create posts']);

// Define a different `create posts` permission for the regular users belonging to the web guard
$permission = Permission::create(['guard_name' => 'web', 'name' => 'create posts']);
```

### Assigning permissions and roles to guard users

You can use the same methods to assign permissions and roles to users as described above in [using permissions and roles](#sing permissions-and-roles). Just make sure the `guard_name`s on the permission or role match the guard of the user, otherwise a `GuardDoesNotMatch` exception will be thrown.

### Using blade directives with multiple guards

You can use all of the blade directives listed in [using blade directives](#using-blade-directives) by passing in the guard you wish to use as the second argument to the directive:

```php
@role('super-admin', 'admin')
    I'm a super-admin!
@else
    I'm not a super-admin...
@endrole
```

### Using blade directives with scoped roles

To use Blade directives with scoped roles, parameters to provide are `role_name, guard_name, restrictable_fully_qualified_class, restrictable_id`.
Most of the time, you want to provide a null `guard_name`, while `restrictable_fully_qualified_class` and 
`restrictable_id` must both be not null, or they will be ignored.

## Using a middleware

The package doesn't include a middleware to check permissions but it's very trivial to add this yourself:

``` bash
$ php artisan make:middleware RoleMiddleware
```

This will create a `app/Http/Middleware/RoleMiddleware.php` file for you, where you can handle your role and permissions check:

```php
use Auth;

// ...

public function handle($request, Closure $next, $role, $permission)
{
    if (Auth::guest()) {
        return redirect($urlOfYourLoginPage);
    }

    if (! $request->user()->hasRole($role)) {
       abort(403);
    }
    
    if (! $request->user()->can($permission)) {
       abort(403);
    }

    return $next($request);
}
```

Don't forget to add the route middleware to `app/Http/Kernel.php` file:

```php
protected $routeMiddleware = [
    // ...
    'role' => \App\Http\Middleware\RoleMiddleware::class,
];
```

Now you can protect your routes using the middleware you just set up:

```php
Route::group(['middleware' => ['role:admin,access_backend']], function () {
    //
});
```

## Extending

If you need to extend or replace the existing `Role` or `Permission` models you just need to 
keep the following things in mind:

- Your `Role` model needs to implement the `Spatie\Permission\Contracts\Role` contract
- Your `Permission` model needs to implement the `Spatie\Permission\Contracts\Permission` contract
- You must publish the configuration with this command:
  ```bash
  $ php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="config"
  ```
  And update the `models.role` and `models.permission` values

### Cache

If you manipulate unscoped permission/role data directly in the database instead of calling the supplied methods, then you will not see the changes reflected in the application, because unscoped role and permission data is cached to speed up performance.

To manually reset the cache for this package, run:
```bash
php artisan cache:forget spatie.permission.cache
```

When you use the supplied methods, such as the following, the cache is automatically reset for you:

```php
// see earlier in the README for how these methods work:
$user->assignRole('writer');
$user->removeRole('writer');
$role->givePermissionTo('edit-articles');
$role->revokePermissionTo('edit-articles');
```


## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Testing

``` bash
composer test
```

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security

If you discover any security related issues, please email [freek@spatie.be](mailto:freek@spatie.be) instead of using the issue tracker.

## Credits

- [Spatie](https://github.com/spatie)
- [Freek Van der Herten](https://github.com/freekmurze)
- [All Contributors](../../contributors)

This package is a modded version of Spatie one, which is in turn heavily based on [Jeffrey Way](https://twitter.com/jeffrey_way)'s awesome [Laracasts](https://laracasts.com) lessons on [permissions and roles](https://laracasts.com/series/whats-new-in-laravel-5-1/episodes/16). His original code can be found [in this repo on GitHub](https://github.com/laracasts/laravel-5-roles-and-permissions-demo).

Special thanks to [Alex Vanderbist](https://github.com/AlexVanderbist) who greatly helped with `v2`.

## Resources

- [How to create a UI for managing the permissions and roles](http://www.qcode.in/easy-roles-and-permissions-in-laravel-5-4/)

## Alternatives

- [JosephSilber/bouncer](https://github.com/JosephSilber/bouncer)
- [Zizaco/entrust](https://github.com/Zizaco/entrust)
- [bican/roles](https://github.com/romanbican/roles)
- [spatie/laravel-permission](https://github.com/spatie/laravel-permission)

## About Spatie

Spatie is webdesign agency in Antwerp, Belgium. You'll find an overview of all our open source projects [on our website](https://spatie.be/opensource).

## About me and Dreamonkey

I work at Dreamonkey Srl, a startup in Scandiano, Italy, as backend and Android developer. You can find a description of who we are [on our website](https://www.dreamonkey.com/) (still no english translation, though).

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
