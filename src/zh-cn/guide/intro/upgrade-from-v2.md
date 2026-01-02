# 从 2.0 版本升级

> 如果你还没有使用过 Yii 2.0，可以跳过本节，直接进入"[入门](../start/prerequisites.md)"
> 章节。

虽然共享一些共同的理念和价值观，Yii3 在概念上与 Yii 2.0 不同。没有简单的升级
路径，因此请先[检查 Yii 2.0 的维护策略和生命周期终止日期](https://www.yiiframework.com/release-cycle)，
并考虑在 Yii3 上启动新项目，同时将现有项目保留在 Yii 2.0 上。

## PHP 要求

Yii3 需要 PHP 8.2 或更高版本。因此，使用了 Yii 2.0 中未使用的语言功能：

- [类型声明](https://www.php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration)
- [返回类型声明](https://www.php.net/manual/en/functions.returning-values.php#functions.returning-values.type-declaration)
- [类常量可见性](https://www.php.net/manual/en/language.oop5.constants.php)
- [命名参数](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments)
- [匿名类](https://www.php.net/manual/en/language.oop5.anonymous.php)
- [::class](https://www.php.net/manual/en/language.oop5.basic.php#language.oop5.basic.class.class)
- [生成器](https://www.php.net/manual/en/language.generators.php)
- [可变函数](https://www.php.net/manual/en/functions.arguments.php#functions.variable-arg-list)
- [只读属性](https://www.php.net/manual/en/language.oop5.properties.php#language.oop5.properties.readonly-properties)
- [只读类](https://www.php.net/manual/en/language.oop5.basic.php#language.oop5.basic.class.readonly)
- [构造函数属性提升](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion)
- [属性](https://www.php.net/manual/en/language.attributes.php)

## 初步重构

在将 Yii 2.0 项目移植到 Yii3 之前对其进行重构是一个好主意。这将使移植
更容易，并且在尚未迁移到 Yii3 的情况下使相关项目受益。

### 使用 DI 代替服务定位器

由于 Yii3 强制你注入依赖，因此最好准备并从使用
服务定位器（`Yii::$app->`）切换到 [DI 容器](https://www.yiiframework.com/doc/guide/2.0/en/concept-di-container)。

如果出于任何原因使用 DI 容器有问题，请考虑将对 `Yii::$app->` 的所有调用移动到控制器
操作和小部件，并手动将依赖从控制器传递到需要它们的地方。

有关该概念的解释，请参阅[依赖注入和容器](../concept/di-container.md)。

### 引入仓储来获取数据

由于 Active Record 不是在 Yii3 中使用数据库的唯一方式，请考虑引入能够
隐藏数据获取细节并将其收集在单一位置的仓储。你以后可以重做它：

```php
final readonly class PostRepository
{
    public function getArchive()
    {
        // ...
    }

    public function getTop10ForFrontPage()
    {
        // ...
    }

}
```

### 将领域层与基础设施分离

如果你有一个丰富而复杂的领域，最好将其与框架提供的基础设施分离，
即所有业务逻辑都要进入框架无关的类。

### 将更多内容移入组件

Yii3 服务在概念上类似于 Yii 2.0 组件，因此将应用程序的可重用部分
移入组件是一个好主意。

## 需要学习的内容

### Docker

默认应用程序模板使用 [Docker](https://www.docker.com/get-started/) 来运行应用程序。
学习如何使用它并将其用于你自己的项目是一个好主意，因为它提供了很多好处：

1. 与生产环境完全相同的环境。
2. 除了 Docker 本身之外不需要安装任何东西。
3. 环境是针对每个应用程序的，而不是针对每个服务器的。

### 环境变量

Yii3 应用程序模板使用[环境变量](https://en.wikipedia.org/wiki/Environment_variable)
来配置应用程序的某些部分。这个概念[对于 Docker 化的应用程序非常方便](https://12factor.net/)，
但对于 Yii 1.1 和 Yii 2.0 的用户来说可能很陌生。

### 处理程序

与 Yii 2.0 不同，Yii3 不使用控制器。相反，它使用[处理程序](../structure/handler.md)，
它们类似于控制器但有所不同。

### 应用程序结构

建议的 Yii3 应用程序结构与 Yii 2.0 不同。
它在[应用程序结构](../structure/overview.md)中描述。

尽管如此，Yii3 是灵活的，因此仍然可以在 Yii3 中使用类似于 Yii 2.0 的结构。
