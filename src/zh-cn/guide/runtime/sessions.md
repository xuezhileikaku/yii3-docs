# 会话

会话在请求之间持久化数据，而无需将它们传递给客户端并返回。
Yii 有[会话包](https://github.com/yiisoft/session)来处理会话数据。

要将其添加到您的应用程序，请使用 composer：

```shell
composer require yiisoft/session
```

## 配置中间件

要在请求之间保持会话，您需要将 `SessionMiddleware` 添加到您的路由组或
应用程序中间件。
当您在同一应用程序中同时拥有基于令牌的身份验证的 API 和
常规 Web 路由时，您应该更喜欢路由组。这样可以避免为 API 端点启动会话。

要为某些路由组添加会话，请像以下那样编辑 `config/routes.php`：

```php
<?php

declare(strict_types=1);

use Yiisoft\Router\Group;
use Yiisoft\Session\SessionMiddleware;

return [
    Group::create('/blog')
        ->middleware(SessionMiddleware::class)
        ->routes(
            // ...
        )
];
```

要为整个应用程序添加会话，请像以下那样编辑 `config/application.php`：

```php
return [
    Yiisoft\Yii\Web\Application::class => [
        '__construct()' => [
            'dispatcher' => DynamicReference::to(static function (Injector $injector) {
                return ($injector->make(MiddlewareDispatcher::class))
                    ->withMiddlewares(
                        [
                            Router::class,
                            CsrfMiddleware::class,
                            SessionMiddleware::class, // <-- 添加这个
                            ErrorCatcher::class,
                        ]
                    );
            }),
        ],
    ],
];
```

## 打开和关闭会话

```php
public function actionProfile(\Yiisoft\Session\SessionInterface $session)
{
    // 如果尚未启动会话，则启动会话
    $session->open();

    // 使用会话

    // 写入会话值，然后关闭它
    $session->close();
}
```

> [!NOTE]
> 尽早关闭会话是一个好习惯，因为许多会话实现在会话打开时会阻止其他
> 请求。

还有两种方法可以关闭会话：

```php
public function actionProfile(\Yiisoft\Session\SessionInterface $session)
{
    // 丢弃更改并关闭会话
    $session->discard();

    // 完全销毁会话
    $session->destroy();
}
```

## 使用会话数据

通常，您将使用以下方法来处理会话数据：

```php
public function actionProfile(\Yiisoft\Session\SessionInterface $session)
{
    // 获取值
    $lastAccessTime = $session->get('lastAccessTime');

    // 获取所有值
    $sessionData = $session->all();

    // 设置值
    $session->set('lastAccessTime', time());

    // 检查值是否存在
    if ($session->has('lastAccessTime')) {
        // ...
    }

    // 删除值
    $session->remove('lastAccessTime');

    // 获取值然后删除它
    $sessionData = $session->pull('lastAccessTime');

    // 从运行时清除会话数据
    $session->clear();
}
```

## Flash 消息

如果您需要某些数据在读取之前保留在会话中，例如在下一页上显示消息的情况，
"flash" 消息就是您所需要的。
Flash 消息是一种特殊类型的数据，仅在当前请求和下一个请求中可用。
之后，它将自动删除。

`FlashInteface` 的使用如下：

```php
/** @var Yiisoft\Session\Flash\FlashInterface $flash */

// 请求 1
$flash->set('warning', 'Oh no, not again.');

// 请求 2
$warning = $flash->get('warning');
if ($warning !== null) {
    // 用它做些什么
}
```

## 自定义会话存储

使用 `Yiisoft\Session\Session` 时，您可以使用自己的存储实现：

```php
$handler = new MySessionHandler();
$session = new \Yiisoft\Session\Session([], $handler);
```

自定义存储必须实现 `\SessionHandlerInterface`。
