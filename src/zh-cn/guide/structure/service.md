# 服务组件

应用程序可能会变得复杂，因此将业务逻辑或基础设施的专注部分提取到服务组件中是有意义的。
它们通常被注入到其他组件或动作处理程序中。这通常通过自动装配完成：

```php
public function actionIndex(CurrentRoute $route, MyService $myService): ResponseInterface
{
    $id = $route->getArgument('id');

    // ...
    $extraData = $myService->getExtraData($id);

    // ...
}
```

Yii3 在技术上并不限制您如何构建服务。通常，不需要从基类继承或实现特定接口：

```php
final readonly class MyService
{
    public function __construct(
        private ExtraDataStorage $extraDataStorage
    )
    {
    }

    public function getExtraData(string $id): array
    {
        return $this->extraDataStorage->get($id);
    }
}
```

服务执行任务或返回数据。它们被创建一次，放入 DI 容器中，然后可以被多次使用。
因此，保持服务无状态是一个好主意，即服务本身及其任何依赖项都不应保持状态。
您可以通过在类级别使用 `readonly` PHP 关键字来确保这一点。

## 服务依赖和配置

服务应始终通过 `__construct()` 定义其对其他服务的所有依赖。这既允许您在
创建服务后立即使用它，也作为服务做太多事情的指标，如果有太多依赖项的话。

- 服务创建后，不应在运行时重新配置它。
- DI 容器实例通常**不应**作为依赖项注入。首选具体接口。
- 在复杂或 "繁重" 的初始化情况下，尝试将其推迟到调用服务方法之前。

对于配置值也是如此。它们应该作为构造函数参数提供。相关的值可以
分组到值对象中。例如，数据库连接通常需要 DSN 字符串、用户名和密码。
这三个可以组合到 Dsn 类中：

```php
final readonly class Dsn
{
    public function __construct(
        public string $dsn,
        public string $username,
        public string $password
    )
    {
        if (!$this->isValidDsn($dsn)) {
            throw new \InvalidArgumentException('提供的 DSN 无效。');
        }
    }

    private function isValidDsn(string $dsn): bool
    {
        // 检查 DSN 有效性
    }
}
```

## 服务方法

服务方法通常做一些事情。它可能是一个完全重复的简单事情，但通常取决于上下文。例如：

```php
final readonly class PostPersister
{
    public function __construct(
        private Storage $db
    )
    {
    }

    public function persist(Post $post)
    {
        $this->db->insertOrUpdate('post', $post);
    }
}
```

有一个服务将文章保存到永久存储（如数据库）中。允许与具体存储通信的对象始终相同，
因此通过构造函数注入，而保存的文章可能不同，因此它作为方法参数传递。

## 所有内容都是服务吗？

通常，选择另一种类类型来放置代码是有意义的。请检查：

- 仓储
- 小部件
- [中间件](middleware.md)
- 实体
