
## A solid way to add multi-language support to your Laravel app


### Using custom middleware to intercept each request and validate URI prefix as a current app locale.

There are often cases when devs need to solve the same problem within different projects.
In the course of my many years of experience, one of those problems has been a localization in Laravel projects. For various projects, I have used some method that I have prepared according to the requirements.

You might think, then why didn’t I make a separate package to solve that problem?
Well, of course, it would be possible to create a package, but usually in the case of different projects, there are so many differences to each other, especially in the front-end part, that making a separate package for only one project becomes ineffective.

So, now I will share the _old-but-gold_ way I have used in many of my Laravel projects, and I hope it may help you.

If you just want to check the ready project now, I will put here the [⭐ GitHub repo](https://github.com/boolfalse/laravel-localization) for the project we about to build.

<img src="https://i.imgur.com/z31JydT.jpg" style="width: 100%;">

For getting well whole lifecycle, let’s [install](https://laravel.com/docs/installation#your-first-laravel-project) a fresh Laravel app.
You can integrate this method in your existing Laravel project as well.

```shell
composer create-project laravel/laravel localization && cd localization
```

In latest Laravel versions **_.env_** file will be created and setup automatically.

Now we have a default Laravel app installed on our machine, so we can quickly test that via built-in opportunity:

```shell
php artisan serve
```

And we can see the result by opening [http://localhost:8000](http://localhost:8000) (by default, 8000 is the port for serving a Laravel app).

Often Laravel app has some routes for customers and some separated protected routes for admins/managers. For the second case we would have to have some URI prefix only for admins/managers, for example: _“dashboard”_.
Let’s assume we need to develop something like we described. For instance, we have

- 3 views/pages/routes for customers: _“home”_, _“about”_, _“contact”_
- and an admin view/page/route: _“dashboard”_

Our **_routes/web.php_** would like this:

```shell
<?php

use Illuminate\Support\Facades\Route;

Route::group([
    'prefix' => '{locale?}',
], function () {
    Route::get('/', function () { return view('pages.home'); })->name('home');
    Route::get('/about', function () { return view('pages.about'); })->name('about');
    Route::get('/contact', function () { return view('pages.contact'); })->name('contact');
});

Route::group([
    'prefix' => '{locale?}/dashboard',
], function () {
    Route::get('/', function () { return view('pages.dashboard'); })->name('dashboard');
});
```

Setup appropriate blade views for testing.

- overwrite **_resources/views/welcome.blade.php_**:

```shell
<!DOCTYPE html>
<html lang="{{ config('localization.locale_lang.' . app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Laravel</title>

    <link href="{{ asset('styles.css') }}" rel="stylesheet">
</head>
<body>

<nav>
    <div class="centered-text">
        <h1>Laravel Localization</h1>
        <p>{{ __('page.hello_world') }}</p>
    </div>
    <ul class="drop-down closed">
        <li class="cursor-pointer">
            <p class="nav-button">Change Language</p>
        </li>
        @foreach(config('localization.locales') as $locale)
            <li data-locale="{{ $locale }}" class="ss-change-locale">
                <a href="javascript:void(0)">
                    {{ __('page.locales.name.' . $locale) }}
                </a>
            </li>
        @endforeach
    </ul>
    <div class="page-content">
        @yield('content')
    </div>
</nav>

<script src="{{ asset('scripts.js') }}"></script>
<script>
    // SWITCH LANGUAGE
    var currentUri = window.location.pathname;
    var locales = "{{ implode('|', config('localization.locales')) }}";
    var currentLocale = "{{ app()->getLocale() }}";
    var elements = document.querySelectorAll('.ss-change-locale');
    for (let i = 0; i < elements.length; i++) {
        elements[i].addEventListener('click', function () {
            let locale = this.getAttribute('data-locale');
            if (locale === currentLocale) {
                return;
            }
            let newUri = currentUri.replace(new RegExp('^\/(' + locales + ')'), '/' + locale);
            let paramsIndex = window.location.href.indexOf('?');
            let params = (paramsIndex !== -1 ? window.location.href.slice(paramsIndex) : '')
            let port = window.location.port ? ':' + window.location.port : '';
            window.location.href = window.location.protocol + '//' + window.location.hostname + port + newUri + params;
        });
    }
</script>

</body>
</html>
```

As you can see, we’re iterating over all the languages (that we have setup in our **_config/localization.php_** configuration file), and list them with LI tags. And at the bottom some JavaScript snippet, which is responsible for picking up the language selected by the user and redirecting to the appropriate page with the appropriate language prefix.
You can easily modify the code snippet to suit your needs as it is just a way to show how it could be used.

Then create **_pages_** directory in the **_resources/views_** directory to have some partials.

- **_resources/views/pages/home.blade.php_** default component for Home:

```shell
@extends('welcome')
@section('content')
    <p class="centered-text">{{ __('page.home') }}</p>
    <div class="centered-text">
        <a href="{{ route('about') }}" class="button">{{ __('page.about') }}</a>
        <a href="{{ route('contact') }}" class="button">{{ __('page.contact') }}</a>
        <a href="{{ route('dashboard') }}" class="button">{{ __('page.dashboard') }}</a>
    </div>
@endsection
```

- **_resources/views/pages/about.blade.php_** component for About:

```shell
@extends('welcome')
@section('content')
    <p class="centered-text">{{ __('page.about') }}</p>
    <div class="centered-text">
        <a href="{{ route('home') }}" class="button">{{ __('page.home') }}</a>
        <a href="{{ route('contact') }}" class="button">{{ __('page.contact') }}</a>
        <a href="{{ route('dashboard') }}" class="button">{{ __('page.dashboard') }}</a>
    </div>
@endsection
```

- **_resources/views/pages/contact.blade.php_** component for Contact:

```shell
@extends('welcome')
@section('content')
    <p class="centered-text">{{ __('page.contact') }}</p>
    <div class="centered-text">
        <a href="{{ route('home') }}" class="button">{{ __('page.home') }}</a>
        <a href="{{ route('about') }}" class="button">{{ __('page.about') }}</a>
        <a href="{{ route('dashboard') }}" class="button">{{ __('page.dashboard') }}</a>
    </div>
@endsection
```

- **_resources/views/pages/dashboard.blade.php_** one more component for Dashboard. Later you can add some blades like this:

```shell
@extends('welcome')
@section('content')
    <p class="centered-text">{{ __('page.dashboard') }}</p>
    <div class="centered-text">
        <a href="{{ route('home') }}" class="button">{{ __('page.home') }}</a>
        <a href="{{ route('about') }}" class="button">{{ __('page.about') }}</a>
        <a href="{{ route('contact') }}" class="button">{{ __('page.contact') }}</a>
    </div>
@endsection
```

Now, for having a little bit of UI, we can use some ready styles from [CodePen](https://codepen.io/dg1234uk/pen/wGyPRP).
I will modify that assets a bit, so you can get those **styles.css** and **scripts.js** files from the **_public_** folder [here](https://github.com/boolfalse/laravel-localization/tree/master/public).

At this point, as we have all the routes and views setup let’s create a config file specifically for the localizations-related stuff, create a custom middleware, register that middleware in the kernel to intercept all the web requests, and add some translations.

- Create **_app/Http/Middleware/Localization.php_**:

```shell
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\App;
use Illuminate\Support\Facades\Redirect;
use Illuminate\Support\Facades\URL;

class Localization
{
    public function handle(Request $request, Closure $next)
    {
        $first_segment = $request->segment(1);
        if (in_array($first_segment, config('localization.locales'))) {
            App::setLocale($first_segment);
            URL::defaults(['locale' => $first_segment]);

            return $next($request);
        } else {
            $fallback_locale = config('app.fallback_locale');
            $segments = $request->segments();
            array_unshift($segments, $fallback_locale);

            return Redirect::to(implode('/', $segments));
        }
    }
}
```

This custom middleware, called **Localization**, is responsible for handling and setting the application’s locale (language) based on the first segment of the URL.

1. It extracts the first segment from the URL. For example, **_“en”_** in _“example.com/**en**/page”_.
2. It checks if this first segment corresponds to a supported locale as defined in the configuration _config('localization.locales')_.
3. If the first segment is a valid locale, it sets the application’s locale using _App::setLocale($first_segment)_, ensuring that the rest of the application operates in that language.
4. It also updates the URL to include the selected locale as a default for future generated URLs using _URL::defaults(['locale' => $first_segment])_.
5. If the first segment is not a valid locale, it redirects the user to the same URL but with the default (fallback) locale by prepending the fallback locale to the URL segments, as defined in **_config/app.php_** _fallback_locale_ field. Of course, don’t forget to clear caches after each configuration change, running _php artisan optimize_.

Shortly, this custom middleware determines the application’s current language (locale) based on the first URL segment, setting it to a valid locale or redirecting to the default locale if the segment is not recognized.

So if we want to change default locale (language) in our app, we need to change the fallback locale in **_config/app.php_**:
_'fallback_locale' => 'en'_ to _'fallback_locale' => '<ANOTHER_LOCALE>'_.

Register the middleware in the web middleware group by adding the Localization class as a last array element of _“web”_ sub-array of _$middlewareGroups_ array:

```shell
protected $middlewareGroups = [
    'web' => [
        // EXISTING ONES
        \App\Http\Middleware\Localization::class, // APPEND THIS HERE
    ],
    'api' => [
        // EXISTING ONES
    ],
];
```

The reason we did that only just for **_web_** middleware group is that we don’t want to affect other existing routes, such as API endpoints which could be presented in our app.

Now, for having dynamic code in our app, we need to have some configuration file, where we can save all our locales (languages). We will create some **_config/localization.php_** specifically for the localizations-related stuff:

```shell
<?php

return [
    'locales' => [
        'en',
        'cn',
    ],
    'locale_lang' => [
        'en' => 'en_US',
        'cn' => 'zh_CN',
    ],
];
```

As you may already notice, in our app we will have English as a default (fallback) language, and Chinese as a secondary language. And we just use _“locales”_ field in our dropdown HTML, to iterate and list all the locales.

Let’s create some translations as well, which we can use in our application.

- Create **_lang/en/page.php_** (for Laravel 9 and old versions create _resources/lang/en/page.php_):

```shell
<?php

return [
    'locales' => [
        'code' => [
            'en' => "EN",
            'cn' => "中国人",
        ],
        'name' => [
            'en' => "English",
            'cn' => "中国人",
        ],
    ],

    'hello_world' => "Hello World",
    'change_language' => "Change Language",

    'home' => "Home",
    'about' => "About",
    'contact' => "Contact",
    'dashboard' => "Dashboard",
];
```

- Create **_lang/cn/page.php_** (for Laravel 9 and old versions create _resources/lang/cn/page.php_).

```shell
<?php

return [
    'locales' => [
        'code' => [
            'en' => "中文",
            'cn' => "CN",
        ],
        'name' => [
            'en' => "English",
            'cn' => "中文",
        ],
    ],

    'hello_world' => "你好世界",
    'change_language' => "更改语言",

    'home' => "家",
    'about' => "关于",
    'contact' => "联系",
    'dashboard' => "仪表板",
];
```

At this point we have all the necessary things setup to have our app working. Just last time clear the caches and run the server:

```shell
# make sure always to refresh all the caches after each config file changes
php artisan optimize

# quick way to run and test our app
php artisan serve
```

We can now check out our app in the browser on [http://localhost:8000](http://localhost:8000)

Make sure you have disabled automatic translation extensions (like Google Translate extension as in the picture):

<img src="https://i.imgur.com/S0dkL5t.png" style="width: 100%;">

That’s all for the setup.
Later, if you want to add some more translations, the one thing you just need to, to add your stuff in the translation files located in the **lang** directory as we have done already.

Sometimes things happen, when we need to add a completely new language to our existing app that will be supported as well as other languages.
The good part of this article is that we have developed a way to dynamically add new locales/languages.
In our test case we will add Armenian as a third language.

The only two steps you need to do to add a new language, that are:

- Add a language code in the localization-specific configuration file (**_config/localization.php_** in our case):

```shell
<?php

return [
    'locales' => [
        'en',
        'cn',
        'am', // NEW LANGUAGE
    ],
    'locale_lang' => [ // OPTIONAL STUFF
        'en' => 'en_US',
        'cn' => 'zh_CN',
        'am' => 'hy_AM', // NEW LANGUAGE
    ],
];
```

- Add the appropriate lang file like we did before. In our case we will create **_lang/am/page.php_** (for Laravel 9 and old versions create _resources/lang/am/page.php_)

```shell
<?php

return [
    'locales' => [
        'code' => [
            'en' => "中文",
            'cn' => "CN",
            'am' => "ՀԱՅ",
        ],
        'name' => [
            'en' => "English",
            'cn' => "中文",
            'am' => "Հայերեն",
        ],
    ],

    'hello_world' => "Բարև աշխարհ",
    'change_language' => "Փոխել լեզուն",

    'home' => "Տուն",
    'about' => "Մասին",
    'contact' => "Կապ",
    'dashboard' => "Վահանակ",
];
```

You optionally may modify your existing lang files as I need to do in my case as you can notice in the GitHub repo, but it depends on your case.
In my case I will do some small additions in *****/en/page.php** and in *****/cn/page.php**:

```shell
<?php

return [
    'locales' => [
        'code' => [
            // EXISTING CODE
            'am' => "ՀԱՅ", // ADDING THIS
        ],
        'name' => [
            // EXISTING CODE
            'am' => "Հայերեն", // ADDING THIS
        ],
    ],
    // EXISTING CODE
];
```

Finally… again, don’t forget to clear the caches by the artisan command.

**That’s it**.

Now we have integrated a custom and solid way to have Laravel multi-language support, as well as the way to add new languages dynamically.

In the end, I will put here the [GitHub repo ⭐](https://github.com/boolfalse/laravel-localization) for the project we built.

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [**GitHub**](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [**boolfalse.com**](https://boolfalse.com/)

Thank you !!!
