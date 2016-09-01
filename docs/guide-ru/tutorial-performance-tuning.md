Оптимизация производительности
==================

Существует много факторов, влияющих на производительность веб-приложения. Какие-то относятся к окружению, какие-то 
к вашему коду, а какие-то к самому Yii. В этом разделе мы перечислим большинство из них и объясним, как можно улучшить 
производительность приложения, регулируя эти факторы.


## Оптимизация окружения PHP <span id="optimizing-php"></span>

Хорошо сконфигурированное окружение PHP очень важно. Для получения максимальной производительности,

- Используйте последнюю стабильную версию PHP. Мажорные релизы PHP могут принести значительные улучшения производительности.
- Включите кеширование байткода в [Opcache](http://php.net/opcache) (PHP 5.5 и старше) или [APC](http://ru2.php.net/apc) 
  (PHP 5.4 и более ранние версии). Кеширование байткода позволяет избежать затрат времени на обработку и подключение PHP 
  скриптов при каждом входящем запросе.

## Отключение режима отладки <span id="disable-debug"></span>

При запуске приложения в производственном режиме, вам нужно отключить режим отладки. Yii использует значение константы
`YII_DEBUG` чтобы указать, следует ли включить режим отладки. Когда режим отладки включен, Yii тратит дополнительное 
время чтобы создать и записать отладочную информацию.

Вы можете разместить следующую строку кода в начале [входного скрипта](structure-entry-scripts.md) чтобы 
отключить режим отладки:

```php
defined('YII_DEBUG') or define('YII_DEBUG', false);
```

> Info: Значение по умолчанию для константы `YII_DEBUG` — false. 
Так что, если вы уверены, что не изменяете значение по умолчанию где-то в коде приложения, можете просто удалить эту 
строку, чтобы отключить режим отладки.
  

## Использование техник кеширования <span id="using-caching"></span>

Вы можете использовать различные техники кеширования чтобы значительно улучшить производительность вашего приложения. 
Например, если ваше приложение позволяет пользователям вводить текст в формате Markdown, вы можете закешировать 
разобранное содержимого Markdown, чтобы избежать разбора одной и той же разметки Markdown неоднократно 
при каждом запросе. Пожалуйста, обратитесь к разделу [Кеширование](caching-overview.md) чтобы узнать о поддержке 
кеширования, которую предоставляет Yii.


## Включение кеширования схемы <span id="enable-schema-caching"></span>

Кэширование схемы - это специальный *тип кеширования*, который должен быть включен при использовании [Active Record](db-active-record.md).
Как вы знаете, Active Record достаточно умен, чтобы обнаружить информацию о схеме (например, имена столбцов, типы столбцов, 
ограничения) таблицы БД без необходимости описывать ее вручную. Active Record получает эту информацию, выполняя 
дополнительные SQL запросы. При включении кэширования схемы, полученная информация о схеме будет сохранена в кэше и 
повторно использована при последующих запросах.

Чтобы включить кеширование схемы, сконфигурируйте [компонент приложения](structure-application-components.md) `cache` 
для хранения информации о схеме и установите [[yii\db\Connection::enableSchemaCache]] в `true` в [конфигурации приложения](concept-configurations.md):

```php
return [
    // ...
    'components' => [
        // ...
        'cache' => [
            'class' => 'yii\caching\FileCache',
        ],
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=mydatabase',
            'username' => 'root',
            'password' => '',
            'enableSchemaCache' => true,

            // Продолжительность кеширования схемы.
            'schemaCacheDuration' => 3600,

            // Название компонента кеша, используемого для хранения информации о схеме
            'schemaCache' => 'cache',
        ],
    ],
];
```


## Объединение и минимизация ресурсов <span id="optimizing-assets"></span>

Сложные веб-страницы часто подключают много CSS и/или JavaScript файлов. Для уменьшения числа HTTP запросов
и общего размера загрузки этих ресурсов, вы должны рассмотреть вопрос об их объединении в один файл и его сжатии.
Это может сильно увеличить скорость загрузки страницы и снизить нагрузку на сервер. Для получения более подробной
информации обратитесь, пожалуйста, к разделу [Ресурсы](structure-assets.md)


## Оптимизация хранилища сессий <span id="optimizing-session"></span>

По умолчанию данные сессий хранятся в файлах. Это удобно для разработки или в маленьких проектах.
Но когда дело доходит до обработки множества параллельных запросов, то лучше использовать более сложные хранилища, 
такие как базы данных. Yii поддерживает различные хранилища "из коробки". 
Вы можете использовать эти хранилища, сконфигурировав компонент `session` в
[конфигурации приложения](concept-configurations.md) как показано ниже,

```php
return [
    // ...
    'components' => [
        'session' => [
            'class' => 'yii\web\DbSession',

            // Установите следующее, если вы хотите использовать компонент БД, с названием
            // отличным от значения по умолчанию 'db'.
            // 'db' => 'mydb',

            // Чтобы перезаписать таблицу сессий, заданную по умолчанию, установите
            // 'sessionTable' => 'my_session',
        ],
    ],
];
```

Приведенная выше конфигурация использует таблицу базы данных для хранения сессионных данных. По умолчанию, используется 
компонент приложения `db` для подключения к базе данных и сохранения сессионных данных в таблице `session`. Вам нужно будет 
создать таблицу `session` заранее:

```sql
CREATE TABLE session (
    id CHAR(40) NOT NULL PRIMARY KEY,
    expire INTEGER,
    data BLOB
)
```

Вы также можете хранить сессионные данные в кеше с помощью [[yii\web\CacheSession]]. Теоретически, вы можете использовать 
любое поддерживаемое [хранилище кеша](caching-data.md#supported-cache-storage). Тем не менее, помните, что некоторые 
хранилища кеша могут *сбрасывать* закешированные данные при достижении лимитов хранилища. По этой причине, вы должны в 
основном использовать хранилища кеша, которые не имеют таких лимитов.

Если на вашем сервере установлен [Redis](http://redis.io/), настоятельно рекомендуется выбрать его в качестве 
хранилища сессий используя [[yii\redis\Session]].


## Оптимизация базы данных <span id="optimizing-databases"></span>

Выполнение запросов к БД и выборки данных часто являются узким местом производительности веб-приложения. 
Хотя использование техник [кэширования данных](caching-data.md) может *смягчить* снижение производительности, оно не 
решает проблему полностью. Когда база данных содержит огромное количество данных, и данные в кэше невалидны, получение 
свежих данных без правильного проектирования БД и запросов может быть чрезмерно ресурсоемкой операцией.

Общей методикой для повышения производительности запросов к БД является создание индексов для тех столбцов таблицы, по которым делается выборка.
Например, если вам нужно найти запись о пользователе по `username`, вам надо создать индекс на `username`. 
Обратите внимание, что в то время как индексирование может сделать SELECT запросы намного быстрее, оно будет замедлять INSERT, UPDATE и DELETE запросы.

Для сложных запросов к БД рекомендуется создавать представления базы данных (views), чтобы сэкономить время подготовки и разбора запросов.

Последнее, хотя и не менее важное: используйте `LIMIT` в ваших `SELECT` запросах. Это позволяет избежать извлечения 
большого количество данных из базы данных и исчерпания памяти, выделенной для PHP.


## Использование обычных массивов <span id="using-arrays"></span>

Хотя [Active Record](db-active-record.md) очень удобно использовать, это не так эффективно, как использование простых 
массивов, когда вам нужно получить большое количество данных из БД. В этом случае, вы можете вызвать `asArray()` при 
использовании Active Record для получения данных, чтобы извлеченные данные были представлены в виде массивов вместо 
громоздких записей Active Record. Например,

```php
class PostController extends Controller
{
    public function actionIndex()
    {
        $posts = Post::find()->limit(100)->asArray()->all();
        
        return $this->render('index', ['posts' => $posts]);
    }
}
```

В приведенном выше коде, `$posts` будет заполнего массивом строк из таблицы. Каждая строка - это обычный массив. Чтобы 
получить доступ к столбцу `title` в i-й строке, вы можете использовать выражение `$posts[$i]['title']`.

Вы также можете использовать [DAO](db-dao.md) для создания запросов и извлечения данных в виде обычных массивов. 


## Оптимизация автозагрузчика Composer <span id="optimizing-autoloader"></span>


Поскольку автозагрузчик Composer'а используется для подключения большого количества файлов сторонних классов, вы должны 
оптимизировать его, выполнив следующую команду:

```
composer dumpautoload -o
```


## Асинхронная обработка данных <span id="processing-data-offline"></span>

Когда запрос включает в себя некоторые ресурсоемкие операции, вы должны подумать о том, чтобы выполнить эти операции 
асинхронно, не заставляя пользователя ожидать их окончания.

Существует два метода асинхронной обработки данных: pull и push. 

В методе pull, всякий раз, когда запрос включает в себя некоторые сложные операции, вы создаете задачу и сохраняете ее в 
постоянном хранилище, таком как база данных. Затем в отдельном процессе (таком как задание cron) получаете эту задачу и 
обрабатываете ее.

Этот метод легко реализовать, но у него есть некоторые недостатки. Например, задачи надо периодически забирать из 
места их хранения. Если делать это слишком редко, задачи будут обрабатываться с большой задержкой, а если слишком часто - 
это будет создавать большие накладные расходы.

В методе push, вы можете использовать очереди сообщений (например, RabbitMQ, ActiveMQ, Amazon SQS, и т.д.) для управления задачами.
Всякий раз, когда новая задача попадает в очередь, это инициирует обработку этой задачи обработчиком.


## Профилирование производительности <span id="performance-profiling"></span>

Вы должны профилировать код, чтобы определить узкие места в производительности и принять соответствующие меры.
Следующие инструменты для профилирования могут оказаться полезными:

- [Отладочный тулбар Yii и отладчик](https://github.com/yiisoft/yii2-debug/blob/master/docs/guide/README.md)
- [Профайлер XDebug](http://xdebug.org/docs/profiler)
- [XHProf](http://www.php.net/manual/en/book.xhprof.php)