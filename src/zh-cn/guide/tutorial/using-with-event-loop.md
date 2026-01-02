# 在事件循环中使用 Yii

正常的 PHP Web 请求执行周期包括设置环境、获取请求、处理它以形成响应
并发送结果。发送响应后，执行被终止并且其上下文丢失。因此，对于进一步的
请求，整个序列被重复。这种方法在开发简便性方面具有很大优势，因为开发人员
不必太关心内存泄漏或正确清理上下文。另一方面，为每个请求初始化所有内容
需要时间，总体上消耗高达 50% 的处理资源。

有一种运行应用程序的替代方法。事件循环。其想法是一次初始化所有可能的内容，
然后使用它处理多个请求。这种方法通常称为事件循环。

有多种工具可用于实现它。特别是，[FrankenPHP](https://frankenphp.dev/)、
[RoadRunner](https://roadrunner.dev/) 和 [Swoole](https://www.swoole.co.uk/)。

## 事件循环的影响

事件循环 worker 基本上如下所示：

```php
initializeContext();
while ($request = getRequest()) {
   $response = process($request);
   emit($response);
}
```

通常，有多个 worker 同时处理请求，就像传统的 php-fpm 一样。

这意味着在开发应用程序时需要考虑更多。

### 处理是阻塞的

Worker 逐个处理请求，当前的处理会阻塞处理下一个请求。这意味着
长时间运行的进程，就像在一般的 PHP 应用程序中一样，应该通过使用队列放入后台。

### 服务和状态

由于事件循环中的上下文在单个 worker 处理的所有请求-响应之间共享，
因此上一个请求对服务状态所做的所有更改可能会影响当前请求。此外，这可能是一个安全问题，
如果一个用户的数据对另一个用户可用。

有两种方法可以处理它。首先，您可以通过使服务无状态来避免状态。PHP 的 `readonly`
关键字可能对此很有用。其次，您可以在请求处理结束时重置服务的状态。
在这种情况下，状态重置器将帮助您：

```php
initializeContext();
$resetter = $container->get(\Yiisoft\Di\StateResetter::class);
while ($request = getRequest()) {
   $response = process($request);
   emit($response);
   $resetter->reset(); // 我们应该在每次请求时重置此类服务的状态。
}
```

## 集成

- [RoadRunner](using-yii-with-roadrunner.md)
- [Swoole](using-yii-with-swoole.md)
