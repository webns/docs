git c0c262cceb0efe495efebcf966e9a5da864f203b

---

# Шаблоны

- [Создание шаблонов](#creating-views)
- [Передача данных в шаблоны](#passing-data-to-views)
    - [Передача данных во все шаблоны](#sharing-data-with-all-views)
- [Вью-композеры](#view-composers)

<a name="creating-views"></a>
## Создание шаблонов

> {tip} Ищете больше информации о том, как писать Blade-шаблоны? Для начала ознакомьтесь с [документацией Blade](/docs/{{version}}/blade).


Шаблоны содержат HTML-разметку вашего приложения и отделяют контроллеры / бизнес-логику от логики отображения данных. Шаблоны расположены в директории `resources/views`. Давайте посмотрим, как выглядит простой шаблон:

    <!-- Шаблон располагается в resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>Привет, {{ $name }}</h1>
        </body>
    </html>

Так как шаблон хранится в `resources/views/greeting.blade.php`, мы можем получить его с помощью хелпера:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

Как видите, первый передаваемый в хелпер `view` аргумент соответствует названию файла шаблона в папке `resources/views`. Вторым аргументом передается массив с данными, которые будут доступны в шаблоне. В данном случае мы передаем переменную `name`, которая отображается с помощью [синтаксиса Blade](/docs/{{version}}/blade).

Шаблоны могут находиться в поддиректориях внутри `resources/views`. Для обращения к ним следует использовать точечную запись (dot notation). Например, если шаблон хранится в `resources/views/admin/profile.blade.php`, то вы можете обратиться к нему таким образом:

    return view('admin.profile', $data);

> {note} Названия директорий с шаблонами не должны содержать символ `.`.

#### Проверка существования шаблона

Если нужно определить существует ли файл шаблона, вы можете использовать фасад `View`. Метод `exists` вернёт `true`, если он существует:

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

#### Использование первого доступного шаблона
Используя метод `first`, вы можете использовать первый шаблон, который существует в указанном массиве шаблонов. Это полезно, если ваше приложение или пакет позволяют настраивать или перезаписывать шаблоны:

    return view()->first(['custom.admin', 'admin'], $data);
    
Вы также можете вызвать данный метод используя фасад `View`:

    use Illuminate\Support\Facades\View;
    
    return View::first(['custom.admin', 'admin'], $data);

<a name="passing-data-to-views"></a>
## Передача данных в шаблоны

Как было показано в предыдущих примерах, вы можете передать массив с данными в шаблон:

    return view('greetings', ['name' => 'Victoria']);

При передаче информации таким способом данные должны представлять собой массив с парами ключ / значение. Внутри вашего шаблона вы можете получить доступ к каждому значению, используя соответствующий ему ключ, такой как `<?php echo $key; ?> `. В качестве альтернативы передаче полного массива данных вспомогательной функции `view`, вы можете использовать метод `with` для добавления отдельных частей данных в шаблоне:

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### Передача данных во все шаблоны

Иногда необходимо передавать часть данных во все шаблоны, которые используются в вашем приложении. Это можно сделать с помощью метода `share` внутри метода `boot` сервис-провайдера. Его можно добавить в провайдер `AppServiceProvider` или создать отдельный провайдер для этого:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }
        
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }

<a name="view-composers"></a>
## Вью-композеры

Вью-композеры (от англ. composer - композитор) — это функции обратного вызова или методы класса, которые вызываются при отображении шаблона. Если у вас есть данные, которые вы хотели бы отправлять в шаблон при каждом его отображении, то композеры могут помочь организовать такую логику в одном месте.

Для примера, зарегистрируем композер внутри [сервис провайдера](/docs/{{version}}/providers). Мы будем использовать фасад `View` для доступа к основному контракту `Illuminate\Contracts\View\Factory`. По умолчанию в Laravel нет папки для хранения вью-композеров. Вы можете создать её там, где посчитаете нужным. Например, можно создать папку `app/Http/View/Composers`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ViewServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }
        
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\View\Composers\ProfileComposer'
            );
    
            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }
    }

> {note} Помните, если вы создаёте новый сервис-провайдер для хранения ваших композеров, то также следует добавить его в массив `providers` в конфигурационном файле `config/app.php`.

Теперь, когда мы зарегистрировали композер, метод `ProfileComposer@compose` будет вызываться при каждом отображении шаблона `profile`. Давайте создадим класс композера:

    <?php

    namespace App\Http\View\Composers;

    use App\Repositories\UserRepository;
    use Illuminate\View\View;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

Перед началом отображения шаблона композер вызовет метод `compose` с экземпляром `Illuminate\View\View` в качестве первого параметра. Вы можете использовать метод `with` для передачи данных в шаблон.

> {tip} Все композеры подключаются через [сервис контейнер](/docs/{{version}}/container), поэтому можно применить любое внедрение зависимостей внутри конструктора композера.

#### Подключение композера к нескольким шаблонам

Вы можете подключить к композеру несколько шаблонов одновременно, передав их в качестве первого аргумента метода `composer`:

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

Метод `composer` также принимает специальный символ `*`, что позволяет подключить его ко всем шаблонам:

    View::composer('*', function ($view) {
        //
    });

#### Создатели шаблонов

Создатели шаблонов очень похожи на композеры, однако они выполняются сразу после инициализации шаблонов, не дожидаясь их отображения. Для регистрации создателя используется метод `creator`:

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
