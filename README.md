# How-to-Check-User-Online-or-Not-in-Laravel-
In this laravel tutorial, you will learn how to check user online status and last seen.  If you’re building social media web app in laravel. At that time, you need to show users status (online/offline) and last seen. So this tutorial will guide you step by step from scratch to implement Laravel determine users status and last seen app

## Laravel Check User Online Status and Last Seen Example
Follow the below steps and implement laravel app, which will be shown the users status (online/offline) and last seen:

|     *    | Activity |
| ------------- | ------------- |
| Step 1:  |  Install New Laravel App |
| Step 2:  |  Add Database Detail |
| Step 3:  |  Generate Auth Scaffolding |
| Step 4:  |  Add Column in User Table |
| Step 5:  |  Create a Middleware |
| Step 6:  |  Register Middleware in Kernel |
| Step 7:  |  Create Controller by Artisan |
| Step 8:  |  Check Online Status in Blade File |
| Step 9:  |  Run Development Server |


## Step 1: Install New Laravel App
In this step, you need to install laravel latest application setup, So open your terminal OR command prompt and run the following command:

```php
composer create-project --prefer-dist laravel/laravel blog 
```

## Step 2: Add Database Details
After successfully install laravel new app. Next, Go to your laravel app root directory and open .env file. Then add database details as follow:

```php
DB_CONNECTION=mysql 
DB_HOST=127.0.0.1 
DB_PORT=3306 
DB_DATABASE=here your database name here
DB_USERNAME=here database username here
DB_PASSWORD=here database password here
```
## Step 3: Generate Auth Scaffolding
In this step, Run the following commands to generate auth scaffolding:


```cmd
# install laravel ui
composer require laravel/ui --dev
# auth scaffolding
php artisan ui vue --auth
# finally run
npm i && npm run dev
```
## Step 4: Add Column in User Table

You need to add one column in the users table called last_seen.

So, navigate database>migrations folder and open create_users_table file. Then update the last_seen column in this file as follow:


```php
public function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->bigIncrements('id');
        $table->string('name');
        $table->string('email')->unique();
        $table->timestamp('email_verified_at')->nullable();
        $table->string('password');
        $table->timestamp('last_seen')->nullable();
        $table->rememberToken();
        $table->timestamps();
    });
}
```
Now migrate the migrations, So run the following command on command prompt:

```php
php artisan migrate
```
## Step 5: Create a Middleware
In this step, Create a middleware named ActivityByUser. So open your command prompt again and run the following command:

```php
php artisan make:middleware ActivityByUser
```

Next, navigate to app>Http>Middleware and open ActivityByUser.php middleware. Then update the following code into your middleware file:


```php
<?php
namespace App\Http\Middleware;
use App\model\User;
use Closure;
use Auth;
use Cache;
use Carbon\Carbon;
class ActivityByUser
{
    public function handle($request, Closure $next)
    {
        if (Auth::check()) {
            $expiresAt = Carbon::now()->addMinutes(1); // keep online for 1 min
            Cache::put('user-is-online-' . Auth::user()->id, true, $expiresAt);
            // last seen
            User::where('id', Auth::user()->id)->update(['last_seen' => (new \DateTime())->format("Y-m-d H:i:s")]);
        }
        return $next($request);
    }
}
```
## Step 6: Register Middleware in Kernel
In this step, navigate app>Http and open Kernel.php. And register ActivityByUser.php middleware in the kernal.php file. You can see as follow:


```php
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        // \Illuminate\Session\Middleware\AuthenticateSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \App\Http\Middleware\ActivityByUser::class,
    ],
    'api' => [
        'throttle:60,1',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
```
## Step 7: Create Controller by Artisan
In this step, you need to create a controller named UserController. So run the following command on command prompt:

```php
php artisan make:controller UserController
```
Next, navigate to app>Http>Controllers and open UserController.php file. Then update the following method into your UserController.php file:



```php
<?php
namespace App\Http\Controllers;
use App\model\User;
use Carbon\Carbon;
use Illuminate\Http\Request;
use Cache;
class UserController extends Controller
{
    /**
     * Show user online status.
     */
    public function userOnlineStatus()
    {
        $users = User::all();
        foreach ($users as $user) {
            if (Cache::has('user-is-online-' . $user->id))
                echo $user->name . " is online. Last seen: " . Carbon::parse($user->last_seen)->diffForHumans() . " <br>";
            else
                echo $user->name . " is offline. Last seen: " . Carbon::parse($user->last_seen)->diffForHumans() . " <br>";
        }
    }
}
```
Open routes>web.php and create a route:web.php

```php
Route::get('/status', 'UserController@userOnlineStatus');
```
## Step 8: Check Online Status in Blade File
You can also easily show online status in the blade file by using the following code:


```php
@if(Cache::has('user-is-online-' . $user->id))
    <span class="text-success">Online</span>
@else
    <span class="text-secondary">Offline</span>
@endif
```
Navigate to resources>views and open home.blade.php. Then update the following code into home.blade.php.


```php
@extends('layouts.app')
@section('content')
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">Users</div>
                    <div class="card-body">
                        @php $users = DB::table('users')->get(); @endphp
                        <div class="container">
                            <table class="table table-bordered">
                                <thead>
                                <tr>
                                    <th>Name</th>
                                    <th>Email</th>
                                    <th>Status</th>
                                    <th>Last Seen</th>
                                </tr>
                                </thead>
                                <tbody>
                                @foreach($users as $user)
                                    <tr>
                                        <td>{{$user->name}}</td>
                                        <td>{{$user->email}}</td>
                                        <td>
                                            @if(Cache::has('user-is-online-' . $user->id))
                                                <span class="text-success">Online</span>
                                            @else
                                                <span class="text-secondary">Offline</span>
                                            @endif
                                        </td>
                                        <td>{{ \Carbon\Carbon::parse($user->last_seen)->diffForHumans() }}</td>
                                    </tr>
                                @endforeach
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
@endsection
```
The above code will show online or offline status and as well as last seen of all users on your blade view file in laravel.

## Step 9: Run Development Server
In this step, use the following php artisan serve command to start your server locally:


```php
 phpartisan serve
```
# Conclusion
In this tutorial, you have learned how to show the online or offline status of users. And as well as last seen of users in laravel apps.





