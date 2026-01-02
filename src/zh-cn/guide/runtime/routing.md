# 路由和 URL 生成

通常，Yii 应用程序使用某些处理器处理某些请求。
它根据请求 URL 选择处理器。
执行此工作的应用程序部分是路由器，选择处理器、实例化它
并运行处理器方法的过程是*路由*。

路由的反向过程是*URL 生成*，它从给定的命名路由
和关联的查询参数创建 URL。
当您稍后请求创建的 URL 时，路由过程可以将其解析回原始路由
和查询参数。

路由和 URL 生成是独立的服务，但它们使用一组公共路由进行 URL 匹配和
URL 生成。

## 配置路由

通过配置路由，您可以让您的应用程序识别任意 URL 格式，而无需修改现有的
应用程序代码。您可以在 `/config/routes.php` 中配置路由。文件的结构如下：

```php
<?php

declare(strict_types=1);

use App\Controller\SiteController;
use Yiisoft\Router\Route;

return [
    Route::get('/')
        ->action([SiteController::class, 'index'])
        ->name('site/index')
];
```

该文件返回路由数组。定义路由时，您从对应于某个
HTTP 请求类型的方法开始：

- get
- post
- put
- delete
- patch
- head
- options

如果您需要多种方法，可以使用 `methods()`：

```php
<?php

declare(strict_types=1);

use App\Controller\SiteController;
use Yiisoft\Http\Method;
use Yiisoft\Router\Route;

return [
    Route::methods([Method::GET, Method::POST], '/user/{id}')
        ->action([SiteController::class, 'user'])
        ->name('site/user')
];
```

所有这些方法都接受路由模式和处理器。
路由模式定义路由器在路由时如何匹配 URL 以及它如何根据路由名称
和参数生成 URL。
您将在本指南后面了解实际的语法。
您可以将处理器指定为：

- [中间件](../structure/middleware.md)类名称。
- 处理器操作（[HandlerClass、handlerMethod] 的数组）。
- 可调用对象。

在处理器操作的情况下，将实例化 `HandlerClass` 类型的类并调用其 `handlerMethod`：

```php
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

final readonly class HandlerClass
{
    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        // ...
    }
}
```

可调用对象按原样调用：

```php
static function (ServerRequestInterface $request, RequestHandlerInterface $next) use ($responseFactory) {
    $response = $responseFactory->createResponse();
    $response->getBody()->write('You are at homepage.');
    return $response;
}
```

对于处理器操作和可调用对象，类型化参数使用传递给路由的依赖
注入容器自动注入。

通过为 `ServerRequestInterface` 和 `RequestHandlerInterface` 进行类型提示来获取当前请求和处理器。
您可以使用 `middleware()` 方法添加额外的处理器来包装主要处理器：

```php
<?php

declare(strict_types=1);

use Yiisoft\Http\Method;
use Yiisoft\Router\Route;

return [
    Route::methods([Method::GET], '/download/{id}')
        ->action([DownloadController::class, 'download'])
        ->name('download/id')
        ->middleware(LimitDownloadRate::class)
];
```

查看["中间件"](../structure/middleware.md)指南以了解有关如何实现中间件的更多信息。

这在分组路由时特别有用：

```php
<?php

declare(strict_types=1);

use Yiisoft\Router\Group;
use Yiisoft\Router\Route;

return [
    Group::create('/api')
        ->middleware(ApiDataWrapper::class)
        ->routes(
            Route::get('/info/v2')
                ->action(ApiInfo::class)
                ->name('api/info/v2')
                ->middleware(FormatDataResponseAsJson::class),
            Route::get('/user')
                ->action([ApiUserController::class, 'index'])
                ->name('api/user/index'),
            Route::get('/user/{login}')
                ->action([ApiUserController::class, 'profile'])
                ->middleware(FormatDataResponseAsJson::class)
                ->name('api/user/profile'),
        )
];
```

路由器在处理任何以 `/api` 开头的 URL 之前执行 `ApiDataWrapper`。

您可以使用 `name()` 方法为路由命名。基于处理器的名称选择路由名称是个好主意。

您可以为路由参数设置默认值。例如：


```php
<?php

declare(strict_types=1);

use App\Controller\SiteController;
use Yiisoft\Http\Method;
use Yiisoft\Router\Route;

return [
    Route::methods([Method::GET, Method::POST], '/user[/{id}]')
        ->action([SiteController::class, 'user'])
        ->name('site/user')
        ->defaults(['id' => '42'])
];
```

此配置将与 `/user` 和 `/user/123` 都匹配。
在这两种情况下，`CurrentRoute` 服务都将包含填充的 `id` 参数。
在第一种情况下，它将是默认的 `42`，在第二种情况下，它将是 `123`。

如果 URL 应对单个主机有效，您可以使用 `host()` 指定它。

## 路由 <span id="routing"></span>

Yii 路由是灵活的，内部它可能使用不同的路由实现。
实际的匹配算法可能有所不同，但基本思想保持不变。

路由器从上到下匹配配置中定义的路由。
如果存在匹配，则不执行进一步匹配，
路由器执行路由处理器以获取响应。
如果根本没有匹配，路由器将处理传递给
[应用程序中间件集](../structure/middleware.md)中的下一个
中间件。

## 生成 URL <span id="generating-urls"></span>

要基于路由生成 URL，路由应该有一个名称：

```php
<?php

declare(strict_types=1);

use App\Controller\TestController;
use Yiisoft\Router\Route;

return [
    Route::get('/test', [TestController::class, 'index'])
        ->name('test/index'),
    Route::post('/test/submit/{id}', [TestController::class, 'submit'])
        ->name('test/submit')
];
```


生成如下所示：

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use Psr\Http\Message\ResponseInterface;
use Yiisoft\Router\UrlGeneratorInterface;

final readonly class TestController extends AbstractController
{
    protected function name(): string
    {
        return 'test';
    }

    public function index(UrlGeneratorInterface $urlGenerator): ResponseInterface
    {
        $url = $urlGenerator->generate('test/submit', ['id' => '42']);
        // ...
    }
}
```

在上述代码中，我们借助[自动依赖注入](../concept/di-container.md)
获取生成器实例，它适用于操作处理器。
在另一个服务中，您可以通过类似的构造函数注入获取实例。
在视图中，URL 生成器可作为 `$url` 使用。

然后我们使用 `generate()` 方法获取实际的 URL。它接受路由名称和命名查询参数数组。
代码将返回 "/test/submit/42"。如果您需要绝对 URL，请改用 `generateAbsolute()`。

## 路由模式 <span id="route-patterns"></span>

使用的路由模式取决于所使用的底层实现。
默认实现是 [nikic/FastRoute](https://github.com/nikic/FastRoute)。

基本模式是静态的，如 `/test`。这意味着它们必须完全匹配才能进行路由匹配。

### 命名参数 <span id="named-parameters"></span>

模式可以包含一个或多个命名参数，这些参数在模式中以
`{ParamName:RegExp}` 的格式指定，其中 `ParamName` 指定参数名称，`RegExp` 是用于匹配参数值的可选正则
表达式。
如果未指定 `RegExp`，则表示参数值应该是不包含任何斜杠的字符串。

> [!NOTE]
> 您只能在参数内使用正则表达式。模式的其余部分被视为纯文本。

您不能使用捕获组。例如 `{lang:(en|de)}` 不是有效的占位符，因为 `()` 是
捕获组。相反，您可以使用 `{lang:en|de}` 或 `{lang:(?:en|de)}`。

在路由匹配时，路由器使用与 URL 的相应部分匹配的值填充关联的请求属性。
当您使用规则创建 URL 时，它将获取所提供参数的值并将它们插入到
声明参数的位置。

让我们使用一些示例来说明命名参数的工作原理。假设您已声明以下三个模式：


1. `'posts/{year:\d{4}}/{category}`
2. `'posts'`
3. `'post/{id:\d+}'`

- `/posts` 匹配第二个模式；
- `/posts/2014/php` 匹配第一个模式。参数是 `year`，其值为 2014，
  `category`，其值为 `php`；
- `/post/100` 匹配第三个模式。`id` 参数值为 100；
- `/posts/php` 不匹配。

生成 URL 时，您应该使用以下参数：

```php
echo $url->generate('first', ['year' => '2020', 'category' => 'Virology']);
echo $url->generate('second');
echo $url->generate('third', ['id' => '42']);
```

### 可选部分 <span id="optional-parts"></span>

您应该使用 `[` 和 `]` 包装可选的模式部分。
例如，`/posts[/{id}]` 模式将匹配
`http://example.com/posts` 和 `http://example.com/posts/42`。
路由器仅在第二种情况下填充 `CurrentRoute` 服务的 `id` 参数。
在这种情况下，您可以指定默认值：

```php
use \Yiisoft\Router\Route;

Route::get('/posts[/{id}]')->defaults(['id' => '1']);
```

可选部分仅在末尾位置支持，不在路由中间支持。
