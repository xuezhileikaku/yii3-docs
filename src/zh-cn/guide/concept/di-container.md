# 依赖注入和容器

## 依赖注入 <span id="dependency-injection"></span>

在 OOP 中有两种重用事物的方法：继承和组合。

继承很简单：

```php
class Cache
{
    public function getCachedValue($key)
    {
        // ..
    }
}

final readonly class CachedWidget extends Cache
{
    public function render(): string
    {
        $output = $this->getCachedValue('cachedWidget');
        if ($output !== null) {
            return $output;
        }
        // ...
    }
}
```

这里的问题是这两个变得不必要地耦合或相互依赖，使它们更脆弱。

处理此问题的另一种方法是组合：

```php
interface CacheInterface
{
    public function getCachedValue($key);
}

final readonly class Cache implements CacheInterface
{
    public function getCachedValue($key)
    {
        // ..
    }
}

final readonly class CachedWidget
{
    public function __construct(
        private CacheInterface $cache
    )
    {
    }

    public function render(): string
    {
        $output = $this->cache->getCachedValue('cachedWidget');
        if ($output !== null) {
            return $output;
        }
        // ...
    }
}
```

我们避免了不必要的继承，并使用接口来减少耦合。你可以替换缓存
实现而无需更改 `CachedWidget`，因此它变得更加稳定。

这里的 `CacheInterface` 是一个依赖：一个对象依赖的另一个对象。
将依赖实例放入对象（`CachedWidget`）的过程称为依赖注入。
有许多方法可以执行它：

- 构造函数注入。最适合强制依赖。
- 方法注入。最适合可选依赖。
- 属性注入。在 PHP 中最好避免，除了数据传输对象。

### 为什么使用私有属性 <span id="why-private-properties"></span>

在上面的组合示例中，请注意 `$cache` 属性被声明为 `private`。

这种方法通过确保对象具有用于交互的明确定义的接口而不是
直接属性访问来拥抱组合，使代码更易于维护，并且不太容易出现某些类型的错误。

此设计选择提供了几个好处：

- **封装**：带有 getter/setter 的私有属性允许你控制访问并在不破坏现有代码的情况下进行未来更改。
- **数据完整性**：Setter 可以在存储值之前验证、规范化或格式化值，确保属性包含有效数据。
- **不可变性**：私有属性支持不可变对象模式，其中 setter `with*()` 方法返回新实例而不是修改当前实例。
- **灵活性**：你可以创建只读或只写属性，或者在以后为属性访问添加其他逻辑。


## DI 容器 <span id="di-container"></span>

注入基本依赖项很简单。你选择一个你不关心依赖项的地方，
通常是操作处理器，你不需要对其进行单元测试，创建所需的依赖实例
并将它们传递给依赖类。

当依赖项总体上很少并且没有嵌套依赖项时，它工作得很好。当有很多
并且每个依赖项本身都有依赖项时，实例化整个层次结构变得一个繁琐的过程，这
需要大量代码，并可能导致难以调试的错误。

此外，许多依赖项（例如某些第三方 API 包装器）对于使用它的任何类都是相同的。
因此，这样做是有意义的：

- 定义如何实例化这样的 API 包装器。
- 在需要时实例化它，并且每个请求仅一次。

这就是依赖容器的作用。

依赖注入（DI）容器是一个知道如何实例化和配置对象及其所有依赖对象的对象。[Martin Fowler 的文章](https://martinfowler.com/articles/injection.html)很好地解释了为什么 DI 容器有用。这里我们主要解释 Yii 提供的 DI 容器的用法。

Yii 通过 [yiisoft/di](https://github.com/yiisoft/di) 包
和 [yiisoft/injector](https://github.com/yiisoft/injector) 包提供 DI 容器功能。

### 配置容器 <span id="configuring-container"></span>

因为要创建新对象你需要其依赖项，所以你应该尽早注册它们。
你可以在应用程序配置 `config/web.php` 中执行此操作。对于以下服务：

```php
final class MyService implements MyServiceInterface
{
    public function __construct(int $amount)
    {
    }

    public function setDiscount(int $discount): void
    {

    }
}
```

配置可以是：

```php
return [
    MyServiceInterface::class => [
        'class' => MyService::class,
        '__construct()' => [42],
        'setDiscount()' => [10],
    ],
];
```

这等同于以下内容：

```php
$myService = new MyService(42);
$myService->setDiscount(10);
```

还有其他声明依赖项的方法：

```php
return [
    // 为接口声明一个类，自动解析依赖项
    EngineInterface::class => EngineMarkOne::class,

    // 数组定义（同上）
    'full_definition' => [
        'class' => EngineMarkOne::class,
        '__construct()' => [42],
        '$propertyName' => 'value',
        'setX()' => [42],
    ],

    // 闭包
    'closure' => static function(ContainerInterface $container) {
        return new MyClass($container->get('db'));
    },

    // 静态调用
    'static_call' => [MyFactory::class, 'create'],

    // 对象实例
    'object' => new MyClass(),
];
```

### 注入依赖项 <span id="injecting-dependencies"></span>

直接在类中引用容器是一个坏主意，因为代码变得非通用，
耦合到容器接口，更糟糕的是，依赖项变得隐藏。
因此，Yii 通过根据方法参数类型在某些构造函数和方法中自动从容器注入对象来反转控制。

这主要在操作处理器的构造函数和处理方法中完成：

```php
use \Yiisoft\Cache\CacheInterface;

final readonly class MyController
{
    public function __construct(
        private CacheInterface $cache
    )
    {
        $this->cache = $cache;
    }

    public function actionDashboard(RevenueReport $report)
    {
        $reportData = $this->cache->getOrSet('revenue_report', function() use ($report) {
            return $report->getData();
        });

        return $this->render('dashboard', [
           'reportData' => $reportData,
        ]);
    }
}
```

由于它是实例化和调用操作处理器的 [yiisoft/injector](https://github.com/yiisoft/injector)，它
检查构造函数和方法参数类型，从容器获取这些类型的依赖项，并将它们作为
参数传递。这通常称为自动装配。对于子依赖项也是如此，如果你没有明确给出依赖项，
容器将首先检查它是否有这样的依赖项。
声明你需要的依赖项就足够了，它将自动从容器获取。


## 参考 <span id="references"></span>

- [Martin Fowler 的控制反转容器和依赖注入模式](https://martinfowler.com/articles/injection.html)
