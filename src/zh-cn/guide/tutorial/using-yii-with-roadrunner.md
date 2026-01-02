# 在 Yii 中使用 RoadRunner

[RoadRunner](https://roadrunner.dev/) 是一个由 Golang 驱动的应用程序服务器，与 PHP 集成良好。它
将它作为 worker 运行，每个 worker 可以处理多个请求。这种操作模式通常称为
[事件循环](using-with-event-loop.md)，允许不为每个请求重新初始化框架，从而显著提高性能。

## 安装

RoadRunner 可在 Linux、macOS 和 Windows 上运行。安装它的最佳方法是使用 Composer：

```
composer require yiisoft/yii-runner-roadrunner
```

安装完成后，运行

```
./vendor/bin/rr get
```

这将下载现成的可用的 RoadRunner 服务器 `rr` 二进制文件。

## 配置

首先，我们需要配置服务器本身。创建 `/.rr.yaml` 并添加以下配置：

```yaml
server:
  command: "php worker.php"

rpc:
  listen: tcp://127.0.0.1:6001

http:
  address: :8080
  pool:
    num_workers: 4
    max_jobs: 64
  middleware: ["static", "headers"]
  static:
    dir:   "public"
    forbid: [".php", ".htaccess"]
  headers:
    response:
      "Cache-Control": "no-cache"

reload:
  interval: 1s
  patterns: [ ".php" ]
  services:
    http:
      recursive: true
      dirs: [ "." ]

logs:
  mode: production
  level: warn
```

我们指定入口脚本是 `worker.php`，端口 8080 上应该有三个 worker，`public` 目录中的
文件是静态文件，除了 `.php` 和 `.htaccess`。此外，我们还发送了一个额外的标头。

创建 `/worker.php`：

```php
<?php

declare(strict_types=1);


use Yiisoft\Yii\Runner\RoadRunner\RoadRunnerApplicationRunner;

ini_set('display_errors', 'stderr');

require_once __DIR__ . '/preload.php';

(new RoadRunnerApplicationRunner(__DIR__, $_ENV['YII_DEBUG'], $_ENV['YII_ENV']))->run();
```

## 启动服务器

要启动服务器，请执行以下命令：

```
./rr serve -d
```

## 关于 worker 作用域

- 每个 worker 的作用域与其他 worker 隔离。内存不共享。
- 单个 worker 为多个请求提供服务，其中作用域是共享的。
- 在事件循环的每次迭代中，每个依赖状态的服务都应该被重置。
