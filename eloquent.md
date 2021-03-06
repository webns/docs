git 8ccbb483ac1b0396e1be5d4dc1832d3525d60f3d

---

# Eloquent: Начало работы

- [Введение](#introduction)
- [Определение моделей](#defining-models)
    - [Принятые соглашения](#eloquent-model-conventions)
    - [Значения по умолчанию](#default-attribute-values)
- [Получение моделей](#retrieving-models)
    - [Коллекции](#collections)
    - [Разделение результата на блоки](#chunking-results)
    - [Продвинутые подзапросы](#advanced-subqueries)
- [Получение одиночных моделей / агрегатных функций](#retrieving-single-models)
    - [Получение агрегатных функций](#retrieving-aggregates)
- [Вставка и изменение моделей](#inserting-and-updating-models)
    - [Вставки](#inserts)
    - [Изменения](#updates)
    - [Массовое назначение](#mass-assignment)
    - [Другие методы создания](#other-creation-methods)
- [Удаление моделей](#deleting-models)
    - [Псевдоудаление](#soft-deleting)
    - [Запрос псевдоудалённых моделей](#querying-soft-deleted-models)
- [Реплицирование моделей](#replicating-models)    
- [Скоупы (scopes) запросов](#query-scopes)
    - [Глобальные скоупы](#global-scopes)
    - [Локальные скоупы](#local-scopes)
- [Сравнение моделей](#comparing-models)    
- [События](#events)
    - [Обсерверы](#observers)

<a name="introduction"></a>
## Введение

Система объектно-реляционного отображения (ORM) Eloquent — реализация шаблона ActiveRecord в Laravel. Каждая таблица имеет соответствующий класс-модель, который используется для работы с таблицей БД. Модели позволяют запрашивать данные из таблиц, а также вставлять в них новые записи.

Прежде чем начать, настройте ваше соединение с БД в `config/database.php`. Подробнее о настройке БД читайте в соответствующей [документации](/docs/{{version}}/database#configuration).

<a name="defining-models"></a>
## Определение моделей

Для начала создадим модель Eloquent. Модели обычно располагаются в директории `app`, но вы можете поместить их в любое место, в котором работает автозагрузчик в соответствии с вашим файлом `composer.json`. Все модели Eloquent наследуют класс `Illuminate\Database\Eloquent\Model`.

Простейший способ создать экземпляр модели — с помощью [Artisan-команды](/docs/{{version}}/artisan) `make:model`:

    php artisan make:model Flight

Если вы хотите создать [миграцию БД](/docs/{{version}}/migrations) при создании модели, используйте параметр `--migration` или `-m`:

    php artisan make:model Flight --migration

    php artisan make:model Flight -m

<a name="eloquent-model-conventions"></a>
### Принятые соглашения

Теперь давайте посмотрим на пример модели `Flight`, который мы будем использовать для получения и хранения информации из таблицы БД `flights`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }


#### Имена таблиц

Заметьте, что мы не указали, какую таблицу Eloquent должен привязать к нашей модели. Если это имя не указано явно, то в соответствии с принятым соглашением будет использовано имя класса в нижнем регистре (snake_case) и во множественном числе. В нашем случае Eloquent предположит, что модель `Flight` хранит свои данные в таблице `flights`. Вы можете указать произвольную таблицу, определив свойство `table` в классе модели:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Таблица, связанная с моделью.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

#### Первичные ключи

Eloquent также предполагает, что каждая таблица имеет первичный ключ с именем `id`. Вы можете определить свойство `$primaryKey` для указания другого имени.

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The primary key associated with the table.
         *
         * @var string
         */
        protected $primaryKey = 'flight_id';
    }

Вдобавок, Eloquent предполагает, что первичный ключ является инкрементным числом, и автоматически приведёт его к типу `int`. Если вы хотите использовать неинкрементный или нечисловой первичный ключ, задайте свойству `$incrementing` вашей модели значение `false`.

    <?php

    class Flight extends Model
    {
        /**
         * Indicates if the IDs are auto-incrementing.
         *
         * @var bool
         */
        public $incrementing = false;
    }

Если ваш ключ не числовой, установите свойство `$keyType` в `'string'`:

    <?php

    class Flight extends Model
    {
        /**
         * The "type" of the auto-incrementing ID.
         *
         * @var string
         */
        protected $keyType = 'string';
    }

#### Отметки времени

По умолчанию Eloquent ожидает наличия в ваших таблицах столбцов `created_at` и `updated_at` , куда Eloquent будет записывать дату создания и дату обновления. Если вы не хотите такого поведения, установите свойство `$timestamps` класса модели в `false`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Определяет необходимость отметок времени для модели.
         *
         * @var bool
         */
        public $timestamps = false;
    }

Если вы хотите изменить формат отметок времени, задайте свойство `$dateFormat` вашей модели. Это свойство определяет, как атрибуты времени будут храниться в базе данных, а также задаёт их формат при сериализации модели в массив или JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Формат хранения отметок времени модели.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

Если вам надо изменить имена столбцов для хранения отметок времени, вы можете задать константы `CREATED_AT` и `UPDATED_AT`:

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }

#### Соединение с БД

По умолчанию модели Eloquent будут использовать дефолтное соединение с БД, настроенное для вашего приложения. Если вы хотите указать другое соединение для модели, используйте свойство `$connection`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Название соединения для модели.
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }

<a name="default-attribute-values"></a>
### Значения по умолчанию

Если вы хотите определить значения по умолчанию для некоторых атрибутов dашей модели, вы можете определить свойство `$attributes`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The model's default values for attributes.
         *
         * @var array
         */
        protected $attributes = [
            'delayed' => false,
        ];
    }

<a name="retrieving-models"></a>
## Получение моделей

После создания модели и [связанной с ней таблицы](/docs/{{version}}/migrations#writing-migrations), вы можете начать получать данные из вашей БД. Каждая модель Eloquent представляет собой мощный [конструктор запросов](/docs/{{version}}/queries), позволяющий удобно выполнять запросы к связанной таблице. Например:

    <?php

    use App\Flight;

    $flights = App\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### Добавление дополнительных ограничений

Метод `all` в Eloquent возвращает все результаты из таблицы модели. Поскольку модели Eloquent работают как [конструктор запросов](/docs/{{version}}/queries), вы можете также добавить ограничения в запрос, а затем использовать метод `get` для получения результатов:

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} Все методы, доступные в [конструкторе запросов](/docs/{{version}}/queries), также доступны при работе с моделями Eloquent. Вы можете использовать любой из них в запросах Eloquent.

#### Обновление данных в модели

Вы можете обновить аттрибуты в модели при помощи методов `fresh` и `refresh`.

Метод `fresh` делает запрос в базу данных и возвращает такую же модель с новыми данными. Существующая модель не изменяется:

    $flight = App\Flight::where('number', 'FR 900')->first();

    $freshFlight = $flight->fresh();

Метод `refresh` перезаписывает текущую модель новыми данными из БД. Все заргуженные отношения обновляются тоже:

    $flight = App\Flight::where('number', 'FR 900')->first();

    $flight->number = 'FR 456';

    $flight->refresh();

    $flight->number; // "FR 900"

<a name="collections"></a>
### Коллекции

Такие методы Eloquent, как `all` и `get` , которые получают несколько результатов, возвращают экземпляр`Illuminate\Database\Eloquent\Collection`. Класс `Collection` предоставляет [множество полезных методов](/docs/{{version}}/eloquent-collections#available-methods) для работы с результатами Eloquent:

    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

Вы также можете просто перебирать такую коллекцию в цикле как массив:

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
### Разделение результата на блоки

Если вам нужно обработать тысячи записей Eloquent, используйте команду `chunk`. Метод `chunk` будет получать заданное количество моделей Eloquent, передавая их для обработки функции во втором аргументе. Использование этого метода уменьшает используемый объём оперативной памяти при работе с большими наборами данных:

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

Первый передаваемый в метод аргумент — число записей, получаемых в одном блоке. Передаваемая в качестве второго аргумента функция-замыкание будет вызываться для каждого блока, получаемого из БД. Для получения каждого блока записей, передаваемого в замыкание, будет выполнен запрос к базе данных.

#### Использование курсоров

Метод `cursor` позволяет проходить по записям базы данных, используя курсор, который выполняет только один запрос. При обработке больших объёмов данных метод `cursor` может значительно уменьшить расход памяти:

    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

Метод `cursor` возвращает экземпляр `Illuminate\Support\LazyCollection`. [Ленивые коллекции](/docs/{{version}}/collections#lazy-collections) похожи на обычные, но загружают в память только один элемент за раз.

    $users = App\User::cursor()->filter(function ($user) {
        return $user->id > 500;
    });

    foreach ($users as $user) {
        echo $user->id;
    }

<a name="advanced-subqueries"></a>
### Продвинутые подзапросы

#### Выборка подзапросами

Eloquent предлагает расширенную поддержку подзапросов, которая позволяет извлекать информацию из связанных таблиц в одном запросе. Например, представим, что у нас есть таблица рейсов `destinations` и таблица рейсов `flights` в пункты назначения. Таблица `flights` содержит столбец `arrived_at`, в котором указано, когда рейс прибыл в пункт назначения.

Используя функцонал методов `select` и `addSelect`, мы можем выбрать все `destinations` и название рейса, который в последний раз прибыл в этот пункт назначения, с помощью одного запроса:

    use App\Destination;
    use App\Flight;

    return Destination::addSelect(['last_flight' => Flight::select('name')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->limit(1)
    ])->get();

#### Сортировка результатов подзапросов

Кроме того, метод построителя запросов `orderBy` тоже поддерживает подзапросы. Мы можем использовать эту функциональность для сортировки всех пунктов назначения в зависимости от того, когда последний рейс прибыл в этот пункт назначения. Опять же, это может быть сделано во время выполнения одного запроса к базе данных:

    return Destination::orderByDesc(
        Flight::select('arrived_at')
            ->whereColumn('destination_id', 'destinations.id')
            ->orderBy('arrived_at', 'desc')
            ->limit(1)
    )->get();


<a name="retrieving-single-models"></a>
## Получение одиночных моделей / агрегатных функций

Кроме получения всех записей указанной таблицы вы можете также получить конкретные записи с помощью `find` или `first`. Вместо коллекции моделей эти методы возвращают один экземпляр модели:

    // Получение модели по её первичному ключу...
    $flight = App\Flight::find(1);

    // Получение первой модели, удовлетворяющей условиям...
    $flight = App\Flight::where('active', 1)->first();

    // Краткая запись предыдущего способа
    $flight = App\Flight::firstWhere('active', 1);

Также вы можете вызвать метод `find` с массивом первичных ключей, который вернёт коллекцию подходящих записей:

    $flights = App\Flight::find([1, 2, 3]);

Иногда вы можете захотеть получить первый результат запроса или выполнить какое-либо другое действие, если результаты не найдены. Метод `firstOr` вернет первый найденный результат или, если результаты не найдены, выполнит заданную функцию. Результат этой функции будет результатом метода `firstOr`:

    $model = App\Flight::where('legs', '>', 100)->firstOr(function () {
            // ...
    });

Метод `firstOr` также может принимать массив столбцов, если вам нужны определённые поля в модели:

    $model = App\Flight::where('legs', '>', 100)
                ->firstOr(['id', 'legs'], function () {
                    // ...
                });    

#### Исключения в случае отсутствия результатов

Иногда вам нужно бросить исключение, если определённая модель не была найдена. Это удобно в роутах и контроллерах. Методы `findOrFail` и `firstOrFail` получают первый результат запроса. А если результатов не найдено, выбрасывается исключение `Illuminate\Database\Eloquent\ModelNotFoundException`:

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();

Если исключение не поймано, пользователю автоматически посылается HTTP-отклик `404`. Нет необходимости писать явные проверки для возврата откликов `404` при использовании этих методов:

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### Получение агрегатных функций

Вы также можете использовать `count`, `sum`, `max` и другие [агрегатные методы](/docs/{{version}}/queries#aggregates), предоставляемые [конструктором запросов](/docs/{{version}}/queries). Эти методы возвращают соответствующее скалярное значение вместо полного экземпляра модели:

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## Вставка и изменение моделей

<a name="inserts"></a>
### Вставки

Для создания новой записи в БД создайте экземпляр модели, задайте атрибуты модели и вызовите метод `save`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Flight;
    use Illuminate\Http\Request;

    class FlightController extends Controller
    {
        /**
         * Создание нового экземпляра рейса.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Проверка запроса...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();
        }
    }

В этом примере мы просто присвоили значение параметра `name` из входящего HTTP-запроса атрибуту `name` экземпляра модели `App\Flight`. При вызове метода `save` запись будет вставлена в таблицу. Отметки времени `created_at` и `updated_at` будут автоматически установлены при вызове `save`, поэтому их не нужно задавать вручную.

<a name="updates"></a>
### Изменения

Метод `save` можно использовать и для изменения существующей модели в БД. Для изменения модели вам нужно получить её, изменить необходимые атрибуты и вызвать метод `save`. Отметка времени `updated_at` будет установлена автоматически, поэтому не надо задавать её вручную:

    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

#### Массовые изменения

Изменения можно выполнить для нескольких моделей, которые соответствуют указанному запросу. В этом примере все рейсы, которые отмечены как `active` и местоназначение `destination` которых равно `San Diego`, будут отмечены как задержанные:

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

Метод `update`  ожидает массив пар столбец/значение, обозначающий, какие столбцы необходимо изменить.

> {note} При использовании массовых изменений Eloquent для изменяемых моделей не будут выбрасываться события `saving`, `saved`, `updating` и `updated`. Это происходит потому, что на самом деле модели вообще не извлекаются при массовом изменении.

#### Контроль изменения данных в модели

Методы `isDirty`, `isClean` и `wasChanged` позволяют узнать, изменялись ли аттрибуты модели.

Метод `isDirty` возвращает `true` если данные в модели изменялись с тех пор, как они были загружены из БД. Вы можете указать явно, какой аттрибут нужно проверить.
Метод `isClean` является противоположностью `isDirty`:

    $user = User::create([
        'first_name' => 'Taylor',
        'last_name' => 'Otwell',
        'title' => 'Developer',
    ]);

    $user->title = 'Painter';

    $user->isDirty(); // true
    $user->isDirty('title'); // true
    $user->isDirty('first_name'); // false

    $user->isClean(); // false
    $user->isClean('title'); // false
    $user->isClean('first_name'); // true

    $user->save();

    $user->isDirty(); // false
    $user->isClean(); // true

Метод `wasChanged` определяет, были ли изменены какие-либо атрибуты при последнем сохранении модели в течение текущего цикла запроса. Вы также можете передать имя атрибута, чтобы посмотреть, был ли изменен тот или иной атрибут:

    $user = User::create([
        'first_name' => 'Taylor',
        'last_name' => 'Otwell',
        'title' => 'Developer',
    ]);

    $user->title = 'Painter';
    $user->save();

    $user->wasChanged(); // true
    $user->wasChanged('title'); // true
    $user->wasChanged('first_name'); // false

<a name="mass-assignment"></a>
### Массовое назначение

Вы можете использовать метод `create` для создания и сохранения модели, передав ему массив с аттрибутами. Метод вернёт добавленную модель. Однако перед этим вам нужно определить либо свойство `fillable`, либо `guarded` в классе модели, так как все модели Eloquent изначально защищены от массового заполнения.

Уязвимость массового заполнения проявляется, когда пользователь передаёт с помощью запроса неподходящий HTTP-параметр, и вы не ожидаете, что этот параметр изменит столбец в вашей БД. Например, злоумышленник может послать в HTTP-запросе параметр `is_admin`, который затем передаётся в метод `create` вашей модели, позволяя пользователю повысить свои привилегии до администратора.

Поэтому, для начала надо определить, для каких атрибутов разрешить массовое назначение. Это делается с помощью свойства модели `$fillable`. Например, давайте разрешим массовое назначение атрибута name нашей модели `Flight`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Атрибуты, для которых разрешено массовое назначение.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Теперь мы можем использовать метод `create` для вставки новой записи в БД. Метод `create` возвращает сохранённый экземпляр модели:

    $flight = App\Flight::create(['name' => 'Flight 10']);

Если у вас уже есть экземпляр модели, вы можете заполнить его массивом атрибутов с помощью метода `fill`:

    $flight->fill(['name' => 'Flight 22']);

#### Защитные атрибуты

В то время как параметр `$fillable` служит "белым списком" атрибутов, для которых разрешено массовое назначение, параметр `$guarded` служит "чёрным списком". Параметр `$guarded` должен содержать массив атрибутов, для которых будет запрещено массовое назначение. Атрибутам, не вошедшим в этот массив, будет разрешено массовое назначение. Обратите внимание, вы должны использовать только один из этих параметров - или `$fillable` или `$guarded`, не оба. 

В данном примере всем атрибутам **кроме `price`** разрешено массовое заполнение:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Атрибуты, для которых запрещено массовое назначение.
         *
         * @var array
         */
        protected $guarded = ['price'];
    }

Чтобы разрешить массовое назначение всем атрибутам, определите свойство `$guarded` как пустой массив:

    /**
     * Атрибуты, для которых запрещено массовое назначение.
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### Другие методы создания

#### `firstOrCreate`/ `firstOrNew`

Есть ещё два метода, используемые для создания моделей с помощью массового заполнения: `firstOrCreate` и `firstOrNew`. Метод `firstOrCreate` пытается найти запись БД, используя аттрибуты, указанные в первом аргументе. Если модель не найдена в БД, запись будет вставлена в БД с атрибутами, указанными в первом и во втором аргументе.

Метод  `firstOrNew`, как и `firstOrCreate`, будет пытаться найти в БД запись, соответствующую указанным атрибутам. Однако если модель не найдена, будет возвращён новый экземпляр модели. Учтите, что эта модель ещё не помещена в БД. Вам надо вызвать метод `save` вручную, чтобы сохранить её:

    // Получить рейс по атрибутам или создать, если он не существует...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

    // Получить рейс по атрибутам или создать его с именем и отложенными атрибутами...
    $flight = App\Flight::firstOrCreate(
        ['name' => 'Flight 10'],
        ['delayed' => 1, 'arrival_time' => '11:30']
    );

    // Получить по имени или образцу...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

    // Получить по имени или образцу, с именем и отложенными атрибутами...
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'],
        ['delayed' => 1, 'arrival_time' => '11:30']
    );

#### `updateOrCreate`

Ещё вы можете столкнуться с ситуациями, когда надо обновить существующую модель или создать новую, если её пока нет. Laravel предоставляет метод `updateOrCreate` для выполнения этой задачи за один шаг. Подобно методу `firstOrCreate`, метод `updateOrCreate` сохраняет модель, поэтому не надо вызывать метод `save()`:

    // Если есть рейс из Oakland в San Diego, установить цену $99.
    // Если подходящей модели не существует, создать новую.
    $flight = App\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99, 'discounted' => 1]
    );

<a name="deleting-models"></a>
## Удаление моделей

Для удаления модели вызовите метод `delete` на её экземпляре:

    $flight = App\Flight::find(1);

    $flight->delete();

#### Удаление существующей модели по ключу

В предыдущем примере мы получили модель из БД перед вызовом метода `delete`. Но если вы знаете первичный ключ модели, вы можете удалить модель, не получая её. Для этого укажите в аргументах id этой модели. Метод также принимает перечисление, массив и коллекцию id. 

    App\Flight::destroy(1);

    App\Flight::destroy(1, 2, 3);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(collect([1, 2, 3]));

#### Удаление модели запросом

Вы также можете выполнить оператор удаления на наборе моделей. В этом примере мы удалим все рейсы, отмеченные неактивными. Подобно массовому обновлению, массовое удаление не вызовет никаких событий для удаляемых моделей:

    $deletedRows = App\Flight::where('active', 0)->delete();

> {note} При использовании массового удаления Eloquent для удаляемых моделей не будут выброшены события `deleting` и `deleted`. Это происходит потому, что на самом деле модели вообще не извлекаются при выполнении оператора удаления.

<a name="soft-deleting"></a>
### Псевдоудаление

Кроме обычного удаления записей из БД Eloquent также может «псевдоудалять» модели. Модель при таком удалении на самом деле остаётся в базе данных, но в БД устанавливается её атрибут `deleted_at`. Если у модели значение `deleted_at` не `null`, значит модель псевдоудалена. Для включения псевдоудаления для модели используйте для неё трейт `Illuminate\Database\Eloquent\SoftDeletes` и добавьте столбец `deleted_at` в свойство `$dates`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;
    }

> {tip} Трейт `SoftDeletes` автоматически конвертирует дату `deleted_at` в экземпляр `DateTime` / `Carbon` , не нужно это указывать явно в `$dates`.    

Разумеется, вам необходимо добавить столбец `deleted_at` в вашу таблицу. Для этого используется хелпер [конструктора таблиц](/docs/{{version}}/migrations):

    Schema::table('flights', function (Blueprint $table) {
        $table->softDeletes();
    });

Теперь когда вы вызовете метод `delete`, поле `deleted_at` будет установлено в значение текущей даты и времени. При запросе моделей, использующих псевдоудаление, "удалённые" модели не будут включены в результат запроса.

Для определения того, удалён ли экземпляр модели, используйте метод `trashed`:

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### Запрос псевдоудалённых моделей

#### Включение псевдоудалённых моделей

Как было сказано, псевдоудалённые модели автоматически исключаются из результатов запроса. Для отображения всех моделей, в том числе удалённых, используйте метод `withTrashed`:

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

Метод `withTrashed` может быть использован в [отношениях](/docs/{{version}}/eloquent-relationships):

    $flight->history()->withTrashed()->get();

#### Получение только псевдоудалённых моделей

Если вы хотите получить **только** псевдоудалённые модели, вызовите метод `onlyTrashed`:

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### Восстановление псевдоудалённых моделей

Иногда необходимо восстановить псевдоудалённую модель. Для восстановления псевдоудалённой модели в активное состояние используется метод `restore`:

    $flight->restore();

Вы также можете использовать его в запросе для быстрого восстановления нескольких моделей. Подобно другим массовым операциям, это не вызовет никаких событий для восстанавливаемых моделей:

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

Как и метод `withTrashed`, метод `restore` можно использовать и в [отношениях](/docs/{{version}}/eloquent-relationships):

    $flight->history()->restore();

#### Перманентное удаление моделей

Иногда вам может потребоваться действительно по-настоящему убрать модель из вашей базы данных. Используйте метод `forceDelete`, чтобы перманентно удалить псевдоудалённую модель из БД:

    // Принудительное удаление одного экземпляра модели...
    $flight->forceDelete();

    // Принудительное удаление всех связанных моделей...
    $flight->history()->forceDelete();

<a name="replicating-models"></a>
## Репликация моделей

Вы можете создать несохраненную копию экземпляра модели, используя метод `replicate`. Это особенно полезно, когда у вас есть экземпляры моделей, которые имеют много одинаковых атрибутов:

    $shipping = App\Address::create([
        'type' => 'shipping',
        'line_1' => '123 Example Street',
        'city' => 'Victorville',
        'state' => 'CA',
        'postcode' => '90001',
    ]);

    $billing = $shipping->replicate()->fill([
        'type' => 'billing'
    ]);

    $billing->save();


<a name="query-scopes"></a>
## Скоупы запросов (query scopes)

<a name="global-scopes"></a>
### Глобальные скоупы

Глобальные скоупы позволяют добавить ограничения во все запросы для данной модели. Собственная функция Laravel [псевдоудаление](#soft-deleting) использует глобальные скоупы, чтобы получать из базы данных только "неудалённые" модели. Написание собственных глобальных скоупы обеспечивает удобный и простой способ наложить определённые ограничения на каждый запрос для конкретной модели.

#### Написание глобальных скоупов

Определите класс, реализующий интерфейс  `Illuminate\Database\Eloquent\Scope`. Этот интерфейс требует реализации одного метода: `apply`. Метод  `apply` может добавить к запросу ограничение `where` при необходимости:

    <?php

    namespace App\Scopes;
    
    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Scope;

    class AgeScope implements Scope
    {
        /**
         * Применение скоупа к данному конструктору запросов Eloquent.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            $builder->where('age', '>', 200);
        }
    }

> {tip} Если ваш глобальный скоуп добавляет столбцы для выбора спецификации запроса, следует использовать метод `addSelect` вместо `select`. Это предотвратит непреднамеренную замену существующей спецификации выбора запроса.

#### Применение глобальных скоупов

Вам надо переопределить метод `boot` данной модели и использовать метод `addGlobalScope`:

    <?php

    namespace App;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * "Загружающий" метод модели.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope(new AgeScope);
        }
    }

После этого запрос к `User::all()` будет создавать следующий SQL:

    select * from `users` where `age` > 200

#### Глобальные скоупы с помощью функций

Также Eloquent позволяет определять глобальные скоупы с помощью анонимных функций, что особенно удобно для простых скоупов, которым не нужен отдельный класс:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * "Загружающий" метод модели.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope('age', function (Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }

#### Удаление глобальных скоупов

Если вы хотите удалить глобальную скоуп для данного запроса, то можете использовать метод `withoutGlobalScope`. Этот метод принимает единственный аргумент — имя класса глобальной скоупа:

    User::withoutGlobalScope(AgeScope::class)->get();

Или, если вы использовали не класс, а функцию:

    User::withoutGlobalScope('age')->get();

Если вы хотите удалить несколько или все глобальные скоупы, то можете использовать метод `withoutGlobalScopes`:

    // Удалить все
    User::withoutGlobalScopes()->get();

    // Удалить некоторые
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>
### Локальные скоупы

Скоупы позволяют вам повторно использовать логику запросов в моделях. Например, если вам часто требуется получать пользователей, которые сейчас "популярны". Для создания скоупа просто начните имя метода с префикса `scope`.

Скоупы всегда должны возвращать экземпляр конструктора запросов:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * Scope a query to only include active users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

#### Использование локального скоупа

Когда скоуп определён, вы можете вызывать эти методы при запросах к модели - просто удаляя префикс `scope`. Вы можете даже сцеплять вызовы разных скоупов, например:

    $users = App\User::popular()->active()->orderBy('created_at')->get();

Для формирования отношения "или" воспользуйтесь анонимной функцией:

    $users = App\User::popular()->orWhere(function (Builder $query) {
        $query->active();
    })->get();

Или вы можете использовать функцию высшего порядка `orWhere`, которая сделает такую же цепочку, но более красиво, без анонимной функции: 

    $users = App\User::popular()->orWhere->active()->get();    

#### Динамические скоупы

Иногда вам может потребоваться определить скоуп, которая принимает параметры. Для этого просто добавьте эти параметры после параметра `$query`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include users of a given type.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @param mixed $type
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

А затем передайте их при вызове метода скоупа:

    $users = App\User::ofType('admin')->get();

<a name="comparing-models"></a>
## Сравнение моделей

Иногда может понадобиться определить, являются ли две модели одинаковыми. Метод `is` возвращает `true` если модели имеют один и тот же первичный ключ, таблицу и подключение к базе данных:

    if ($post->is($anotherPost)) {
        //
    }

<a name="events"></a>
## События

Модели Eloquent запускают несколько событий, позволяя вам вставить свой код в следующие моменты жизненного цикла модели: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. События позволяют легко выполнять код каждый раз при сохранении или обновлении в базе данных конкретного класса модели. Каждое событие получает экземпляр модели через ее конструктор.

Событие `retrieved` будет срабатывать, когда существующая модель будет извлечена из базы данных. При первом сохранении новой модели будут происходить события `creating` и `created`. Если модель уже существовала в базе данных и вызывается метод `save`, то будут запущены события `updating` / `updated`. В обоих случаях будут запущены события `saving` / `saved`.

> {note} При массовом обновлении или удалении через Eloquent, события `saved`, `updated`, `deleting` и `deleted` не будут вызваны. Это происходит потому, что при массовом обновлении или удалении модель фактически не извлекается из БД.

Для начала определите свойство `$dispatchesEvents` в своей Eloquent-модели, которая сопоставляет различные моменты жизненного цикла модели Eloquent с вашими собственными [классами событий](/docs/{{version}}/events):

    <?php

    namespace App;

    use App\Events\UserDeleted;
    use App\Events\UserSaved;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The event map for the model.
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

После определения событий Eloquent, вы можете использовать [слушателей событий](/docs/{{version}}/events#defining-listeners) для обработки событий.

<a name="observers"></a>
### Обсерверы (наблюдатели)

#### Определение обсерверов

Если вы прослушиваете много событий для определённой модели, вы можете использовать обверверы (observers) для объединения всех слушателей в единый класс. В таких класах названия методов отражают те события Eloquent, которые вы хотите прослушивать. Каждый такой метод получает модель в качестве единственного аргумента. 

Artisan-команда `make:observer` создаст файл обсервера в папке `App/Observers`:

    php artisan make:observer UserObserver --model=User

Будет создан класс такого вида:

    <?php

    namespace App\Observers;

    use App\User;

    class UserObserver
    {
        /**
         * Handle the User "created" event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }

        /**
         * Handle the User "updated" event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function updated(User $user)
        {
            //
        }

        /**
         * Handle the User "deleted" event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function deleted(User $user)
        {
            //
        }

        /**
         * Handle the User "forceDeleted" event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function forceDeleted(User $user)
        {
            //
        }
    }

Для регистрации обсервера используйте метод `observe` на модели, которую хотите наблюдать. Вы можете зарегистрировать обсерверы в методе `boot` одного из ваших сервис-провайдеров. В этом примере мы зарегистрируем его в `AppServiceProvider`:

    <?php

    namespace App\Providers;

    use App\Observers\UserObserver;
    use App\User;
    use Illuminate\Support\ServiceProvider;

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
            User::observe(UserObserver::class);
        }
    }
