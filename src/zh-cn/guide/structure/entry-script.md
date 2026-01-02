# 入口脚本

入口脚本是应用程序引导过程的第一步。应用程序（Web 应用程序或控制台应用程序）具有单个入口脚本。最终用户向入口脚本发出请求，入口脚本实例化应用程序实例并将请求转发给它们。

Web 应用程序的入口脚本必须存储在 Web 可访问的目录下，以便最终用户可以访问它们。它们通常命名为 `index.php`，但也可以使用任何其他名称，只要 Web 服务器可以找到它们。

控制台应用程序的入口脚本是 `./yii`。

入口脚本主要借助 `ApplicationRunner` 执行以下工作：

* 注册 [Composer 自动加载器](https://getcomposer.org/doc/01-basic-usage.md#autoloading)；
* 获取配置；
* 使用配置初始化依赖注入容器；
* 获取请求实例；
* 将其传递给 `Application` 进行处理并从中获取响应。
* 借助发射器将响应对象转换为发送到客户端浏览器的实际 HTTP 响应。

## Web 应用程序 <span id="web-applications"></span>

以下是应用程序模板的入口脚本中的代码：

```php
<?php

declare(strict_types=1);

use App\ApplicationRunner;

// PHP 内置服务器路由。
if (PHP_SAPI === 'cli-server') {
    // 按原样提供静态文件。
    if (is_file(__DIR__ . $_SERVER['REQUEST_URI'])) {
        return false;
    }

    // 为带点的 URL 显式设置。
    $_SERVER['SCRIPT_NAME'] = '/index.php';
}

require_once dirname(__DIR__) . '/vendor/autoload.php';

$runner = new ApplicationRunner();
// 开发模式：
$runner->debug();
// 运行应用程序：
$runner->run();
```


## 控制台应用程序 <span id="console-applications"></span>

同样，以下是控制台应用程序的入口脚本的代码：

```php
#!/usr/bin/env php
<?php

declare(strict_types=1);

use Psr\Container\ContainerInterface;
use Yiisoft\Config\Config;
use Yiisoft\Di\Container;
use Yiisoft\Di\ContainerConfig;
use Yiisoft\Yii\Console\Application;
use Yiisoft\Yii\Console\Output\ConsoleBufferedOutput;

define('YII_ENV', getenv('env') ?: 'production');

require_once 'vendor/autoload.php';

$config = new Config(
    __DIR__,
    '/config/packages',
);

$containerConfig = ContainerConfig::create()
    ->withDefinitions($config->get('console'))
    ->withProviders($config->get('providers-console'));
$container = new Container($containerConfig);

/** @var ContainerInterface $container */
$container = $container->get(ContainerInterface::class);

$application = $container->get(Application::class);
$exitCode = 1;

try {
    $application->start();
    $exitCode = $application->run(null, new ConsoleBufferedOutput());
} catch (\Error $error) {
    $application->renderThrowable($error, new ConsoleBufferedOutput());
} finally {
    $application->shutdown($exitCode);
    exit($exitCode);
}
```

## 替代运行时

对于 RoadRunner 或 Swoole 等替代运行时，应使用特殊的入口脚本。请参阅：

- [在 RoadRunner 中使用 Yii](../tutorial/using-yii-with-roadrunner.md)
- [在 Swoole 中使用 Yii](../tutorial/using-yii-with-swoole.md)
