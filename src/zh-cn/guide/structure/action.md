# 动作

在 Web 应用程序中，请求 URL 决定了执行的内容。匹配由配置了多个路由的路由器进行。
每个路由都可以附加到一个中间件，该中间件在给定请求的情况下产生响应。
由于中间件整体可以被链接并且可以将实际处理传递给下一个中间件，
我们将实际执行工作的中间件称为动作。

有多种方法可以描述动作。最简单的一种是使用闭包：

```php
use \Psr\Http\Message\ServerRequestInterface;
use \Psr\Http\Message\ResponseInterface;
use Yiisoft\Router\Route;

Route::get('/')->action(function (ServerRequestInterface $request) use ($responseFactory): ResponseInterface {
    $response = $responseFactory->createResponse();
    $response->getBody()->write('您位于主页。');
    return $response;
});
```

这对于简单处理来说是可以的，因为任何更复杂的处理都需要获取依赖项，
所以将处理移动到类方法是个好主意。可以使用回调中间件来实现：

```php
use Yiisoft\Router\Route;

Route::get('/')->action([FrontPageAction::class, 'run']),
```

类本身将如下所示：

```php
use \Psr\Http\Message\ServerRequestInterface;
use \Psr\Http\Message\ResponseInterface;

final readonly class FrontPageAction
{
    public function run(ServerRequestInterface $request): ResponseInterface
    {
        // 为首页构建响应
    }
}
```

在许多情况下，将多个路由的处理分组到一个类中是有意义的：


```php
use Yiisoft\Router\Route;

Route::get('/post/index')->action([PostController::class, 'actionIndex']),
Route::get('/post/view/{id:\d+}')->action([PostController::class, 'actionView']),
```

类本身将如下所示：

```php
use \Psr\Http\Message\ServerRequestInterface;
use \Psr\Http\Message\ResponseInterface;

final readonly class PostController
{
    public function actionIndex(ServerRequestInterface $request): ResponseInterface
    {
        // 渲染文章列表
    }


    public function actionView(ServerRequestInterface $request): ResponseInterface
    {
        // 渲染单篇文章
    }
}
```

我们通常将这样的类称为 "控制器"。

## 自动装配

动作类的构造函数和动作方法都自动从依赖注入容器获取服务：

```php
use \Psr\Http\Message\ServerRequestInterface;
use \Psr\Http\Message\ResponseInterface;
use Psr\Log\LoggerInterface;

final readonly class PostController
{
    public function __construct(
        private PostRepository $postRepository
    )
    {
    }

    public function actionIndex(ServerRequestInterface $request, LoggerInterface $logger): ResponseInterface
    {
        $logger->debug('渲染文章列表');
        // 渲染文章列表
    }


    public function actionView(ServerRequestInterface $request): ResponseInterface
    {
        // 渲染单篇文章
    }
}
```

在上面的示例中，`PostRepository` 通过构造函数自动注入。这意味着它在每个动作中都可用。
Logger 仅注入到 `index` 动作中。


