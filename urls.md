---
git: 83761449ff271ccda95e4ea87eca0f5a772f59df
---

# Генерация URL-адресов

<a name="introduction"></a>
## Введение

Laravel предлагает несколько функций, которые помогут вам в создании URL-адресов для вашего приложения. Эти помощники в первую очередь полезны при построении ссылок в ваших шаблонах и ответах API или при создании ответов-перенаправлений в другую часть вашего приложения.

<a name="the-basics"></a>
## Основы

<a name="generating-urls"></a>
### Создание URL

Помощник `url` используется для генерации произвольных URL-адресов для вашего приложения. Сгенерированный URL-адрес будет автоматически использовать схему (HTTP или HTTPS) и хост из текущего запроса, обрабатываемого приложением:

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### Доступ к текущему URL

Если не передан путь помощнику `url`, то возвращается экземпляр `Illuminate\Routing\UrlGenerator`, позволяющий вам получить доступ к информации о текущем URL:

    // Получить текущий URL без строки запроса ...
    echo url()->current();

    // Получить текущий URL, включая строку запроса ...
    echo url()->full();

    // Получить полный URL-адрес предыдущего запроса ...
    echo url()->previous();

К каждому из этих методов также можно получить доступ через [фасад](/docs/{{version}}/facades) `URL`:

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## URL для именованных маршрутов

Помощник `route` используется для генерации URL-адресов для [именованных маршрутов](/docs/{{version}}/routing#named-routes). Именованные маршруты позволяют создавать URL-адреса без привязки к фактическому URL-адресу, определенному в маршруте. Следовательно, если URL-адрес маршрута изменится, никаких изменений в ваши вызовы функции `route` вносить не нужно. Например, представьте, что ваше приложение содержит маршрут, определенный следующим образом:

    Route::get('/post/{post}', function (Post $post) {
        // ...
    })->name('post.show');

Чтобы сгенерировать URL-адрес этого маршрута, вы можете использовать помощник `route` следующим образом:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

Конечно, помощник `route` также может использоваться для генерации URL-адресов для маршрутов с несколькими параметрами:

    Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
        // ...
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

Любые дополнительные элементы массива, не соответствующие параметрам определения маршрута, будут добавлены в строку запроса URL:

    echo route('post.show', ['post' => 1, 'search' => 'rocket']);

    // http://example.com/post/1?search=rocket

<a name="eloquent-models"></a>
#### Модели Eloquent

Вы часто будете генерировать URL-адреса, используя ключ маршрута (обычно первичный ключ) [модели Eloquent](/docs/{{version}}/eloquent). По этой причине вы можете передавать модели Eloquent в качестве значений параметров. Помощник `route` автоматически извлечет ключ маршрута модели:

    echo route('post.show', ['post' => $post]);

<a name="signed-urls"></a>
### Подписанные URL

Laravel позволяет вам легко создавать «подписанные» URL-адреса для именованных маршрутов. Эти URL-адреса имеют хеш «подписи», добавленный к строке запроса, который позволяет Laravel проверять, что URL-адрес не был изменен с момента его создания. Подписанные URL-адреса особенно полезны для маршрутов, которые общедоступны, но требуют уровня защиты от манипуляций с URL-адресами.

Например, вы можете использовать подписанные URL-адреса для реализации общедоступной ссылки «отказаться от подписки», которая отправляется вашим клиентам по электронной почте. Чтобы создать подписанный URL для именованного маршрута, используйте метод `signedRoute` фасада `URL`:

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

Вы можете исключить домен из хеша подписанного URL, предоставив аргумент `absolute` методу `signedRoute`:

    return URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);

Если вы хотите сгенерировать временный подписанный URL-адрес маршрута, срок действия которого истекает по истечении определенного времени, вы можете использовать метод `temporarySignedRoute`. Когда Laravel проверяет временный подписанный URL-адрес маршрута, он гарантирует, что метка времени истечения срока, закодированная в подписанный URL-адрес, не истекла:

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

<a name="validating-signed-route-requests"></a>
#### Проверка запросов подписанного маршрута

Чтобы убедиться, что входящий запрос имеет действительную подпись, вы должны вызвать метод `hasValidSignature` для входящего объекта запроса `Illuminate\Http\Request`:

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

Иногда может потребоваться разрешить фронтенду вашего приложения добавлять данные к подписанному URL, например, при выполнении пагинации на стороне клиента. Поэтому вы можете указать параметры запроса, которые следует игнорировать при проверке подписанного URL, используя метод `hasValidSignatureWhileIgnoring`. Помните, что игнорирование параметров позволяет любому изменять эти параметры в запросе:

    if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
        abort(401);
    }

Вместо проверки подписанных URL-адресов с помощью экземпляра входящего запроса, вы можете назначить [последника (middleware)](/docs/{{version}}/middleware) `Illuminate\Routing\Middleware\ValidateSignature` маршруту.
Если оно еще не присутствует, вы можете назначить этому посреднику псевдоним в массиве `$middlewareAliases` вашего HTTP ядра:

    /**
     * The application's middleware aliases.
     *
     * Aliases may be used to conveniently assign middleware to routes and groups.
     *
     * @var array<string, class-string|string>
     */
    protected $middlewareAliases = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

После того как вы зарегистрировали посредников в ядре приложения, вы можете назначить его маршруту. Если входящий запрос не имеет действительной подписи, посредник автоматически вернет HTTP-ответ `403`:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

Если ваши подписанные URL-адреса не включают домен в хеш URL, вы должны предоставить аргумент `relative` промежуточному программному обеспечению:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed:relative');

<a name="responding-to-invalid-signed-routes"></a>
#### Ответ на недействительные подписанные маршруты

Когда кто-то посещает подписанный URL-адрес, срок действия которого истек, он получит общую страницу с ошибкой для кода состояния `403` HTTP. Однако вы можете настроить это поведение, определив пользовательское «отображаемое» замыкание для исключения `InvalidSignatureException` в обработчике исключений. Это замыкание должно вернуть HTTP-ответ:

    use Illuminate\Routing\Exceptions\InvalidSignatureException;

    /**
     * Зарегистрируйте обратные вызовы обработки исключений для приложения.
     */
    public function register(): void
    {
        $this->renderable(function (InvalidSignatureException $e) {
            return response()->view('error.link-expired', [], 403);
        });
    }

<a name="urls-for-controller-actions"></a>
## URL для действий контроллера

Функция `action` генерирует URL-адрес для переданного действия контроллера:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Если метод контроллера принимает параметры маршрута, вы можете передать ассоциативный массив параметров маршрута в качестве второго аргумента функции:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="default-values"></a>
## Значения по умолчанию

Для некоторых приложений вы можете указать значения по умолчанию для определенных параметров URL-адреса. Например, представьте, что многие из ваших маршрутов определяют параметр `{locale}`:

    Route::get('/{locale}/posts', function () {
        // ...
    })->name('post.index');

Обременительно передавать `locale` каждый раз при вызове помощника `route`. Итак, вы можете использовать метод `URL::defaults`, чтобы определить значение по умолчанию для этого параметра, которое всегда будет применяться во время текущего запроса. Вы можете вызвать этот метод из [посредника маршрута](/docs/{{version}}/middleware#assigning-middleware-to-routes), чтобы получить доступ к текущему запросу:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\URL;
use Symfony\Component\HttpFoundation\Response;

class SetDefaultLocaleForUrls
{
    /**
     * Обработка входящего запроса.
     *
     * @param \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```
После установки значения по умолчанию для параметра `locale` вам больше не потребуется передавать его значение при генерации URL-адресов с помощью помощника `route`.

<a name="url-defaults-middleware-priority"></a>
#### Параметры URL по умолчанию и приоритет посредника

Установка значений URL по умолчанию может мешать Laravel обрабатывать неявные привязки модели. Следовательно, необходимо [установить приоритет посреднику](/docs/{{version}}/middleware#sorting-middleware), который задает значения URL по умолчанию, и должен выполняться перед посредником Laravel `SubstituteBindings`. Вы можете добиться этого, убедившись, что ваш посредник находится перед посредником `SubstituteBindings` в свойстве `$middlewarePriority` HTTP-ядра вашего приложения.

Свойство `$middlewarePriority` определено в базовом классе `Illuminate\Foundation\Http\Kernel`. Вы можете скопировать его определение из этого класса и перезаписать его в HTTP-ядре вашего приложения, чтобы изменить приоритет:

    /**
     * Список посредников, отсортированный по приоритетности.
     *
     * Заставит неглобальных посредников всегда быть в заданном порядке.
     *
     * @var array
     */
    protected $middlewarePriority = [
        // ...
         \App\Http\Middleware\SetDefaultLocaleForUrls::class,
         \Illuminate\Routing\Middleware\SubstituteBindings::class,
         // ...
    ];
