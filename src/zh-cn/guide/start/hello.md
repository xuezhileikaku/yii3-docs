# Say Hello

本节介绍如何在应用程序中创建一个新的"Hello"页面。这是一个简单的页面，会回显你传递给它的任何内容，如果没有传递任何内容，只会说"Hello!"。

为了实现这个目标，你将定义一个路由并创建[一个处理程序](../structure/handler.md)来完成工作并形成响应。然后你将改进它以使用[视图](../views/view.md)来构建响应。

通过本教程，你将学习三件事：

1. 如何创建处理程序来响应请求。
2. 如何将 URL 映射到处理程序。
3. 如何使用[视图](../views/view.md)来组合响应的内容。

## 创建处理程序 <span id="creating-handler"></span>

对于"Hello"任务，你将创建一个处理程序类，从请求中读取 `message` 参数并将该消息显示回用户。如果请求未提供 `message` 参数，操作将显示默认的"Hello"消息。

创建 `src/Web/Echo/Action.php`：

```php
<?php

declare(strict_types=1);

namespace App\Web\Echo;

use Psr\Http\Message\ResponseFactoryInterface;
use Psr\Http\Message\ResponseInterface;
use Yiisoft\Html\Html;
use Yiisoft\Router\HydratorAttribute\RouteArgument;

final readonly class Action
{
    public function __construct(
        private ResponseFactoryInterface $responseFactory,
    ) {}

    public function __invoke(
        #[RouteArgument('message')]
        string $message = 'Hello!'
    ): ResponseInterface
    {
        $response = $this->responseFactory->createResponse();
        $response->getBody()->write('The message is: ' . Html::encode($message));
        return $response;
    }
}
```

在你的示例中，`__invoke` 方法接收 `$message` 参数，借助 `RouteArgument` 属性从 URL 获取消息。该值默认为 `"Hello!"`。如果向 `/say/Goodbye` 发出请求，操作会将值"Goodbye"分配给 `$message` 变量。

应用程序通过[中间件堆栈](../structure/middleware.md)将响应传递给将响应输出给最终用户的发射器。

## 配置路由器

现在，要将处理程序映射到 URL，你需要在 `config/common/routes.php` 中添加一个路由：

```php
<?php

declare(strict_types=1);

use App\Web;
use Yiisoft\Router\Group;
use Yiisoft\Router\Route;

return [
    Group::create()
        ->routes(
            Route::get('/')
                ->action(Web\HomePage\Action::class)
                ->name('home'),
            Route::get('/say[/{message}]')
                ->action(Web\Echo\Action::class)
                ->name('echo/say'),
        ),
];
```

在上面，你将 `/say[/{message}]` 模式映射到 `\App\Web\Echo\Action`。对于请求，路由器创建一个实例并调用 `__invoke()` 方法。模式的 `{message}` 部分将在此位置指定的任何内容写入 `message` 请求属性。`[]` 将模式的这部分标记为可选。

你还为这个路由指定了 `echo/say` 名称，以便能够生成指向它的 URL。

## 试用 <span id="trying-it-out"></span>

创建操作和视图后，在浏览器中打开 `http://localhost/say/Hello+World`。

此 URL 显示一个页面，其中包含"The message is: Hello World"。

如果在 URL 中省略 `message` 参数，页面将显示"The message is: Hello!"。

## 创建视图模板 <span id="creating-view-template"></span>

通常，任务比打印"hello world"更复杂，并且涉及呈现一些复杂的 HTML。对于此任务，使用视图模板非常方便。它们是你编写用来生成响应主体的脚本。

对于"Hello"任务，创建一个 `src/Web/Echo/template.php` 模板，打印从操作方法接收的 `message` 参数：

```php
<?php
use Yiisoft\Html\Html;
/* @var string $message */
?>

<p>The message is: <?= Html::encode($message) ?></p>
```

在上面的代码中，`message` 参数在打印之前使用 HTML 编码。你需要这样做，因为参数来自最终用户，并且容易通过在参数中嵌入恶意 JavaScript 而受到[跨站脚本 (XSS) 攻击](https://en.wikipedia.org/wiki/Cross-site_scripting)。

当然，你可以在 `say` 视图中放置更多内容。内容可以由 HTML 标签、纯文本甚至 PHP 语句组成。实际上，视图服务将 `say` 视图作为 PHP 脚本执行。

要使用视图，你需要更改 `src/Web/Echo/Action.php`：

```php
<?php

declare(strict_types=1);

namespace App\Web\Echo;

use Psr\Http\Message\ResponseInterface;
use Yiisoft\Router\HydratorAttribute\RouteArgument;
use Yiisoft\Yii\View\Renderer\ViewRenderer;

final readonly class Action
{
    public function __construct(
        private ViewRenderer $viewRenderer,
    ) {}

    public function __invoke(
        #[RouteArgument('message')]
        string $message = 'Hello!'
    ): ResponseInterface
    {
        return $this->viewRenderer->render(__DIR__ . '/template', [
            'message' => $message,
        ]);
    }
}
```

现在打开你的浏览器并再次检查它。你应该看到类似的文本，但应用了布局。

此外，你分离了它如何工作以及它如何呈现的部分。在较大的应用程序中，这有助于处理复杂性。

## 总结 <span id="summary"></span>

在本节中，你接触了典型 Web 应用程序的处理程序和模板部分。你创建了一个作为类一部分的处理程序来处理特定请求。你还创建了一个视图来组合响应的内容。在这个简单的示例中，不涉及数据源，因为使用的唯一数据是 `message` 参数。

你还了解了 Yii 中的路由，它充当用户请求和处理程序之间的桥梁。

在下一节中，你将学习如何获取数据并添加一个包含 HTML 表单的新页面。
