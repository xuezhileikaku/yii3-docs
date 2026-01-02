# 015 — PhpStorm 元数据和属性

## PhpStorm 元数据

[PhpStorm 元数据](https://www.jetbrains.com/help/phpstorm/ide-advanced-metadata.html) 帮助 IDE 在常规类型和 PHPDoc 标记没有帮助的情况下更好地理解代码。

我们为元数据使用以下一组编码风格。

### 元数据位置

- 元数据应该放在 `/.phpstorm.meta.php` 目录中。
- 配置应该拆分为文件。每个文件应该以其配置的类命名。

> [!NOTE]
> PhpStorm 尚不支持子目录。

### 常量

所有常量字典应命名为 `{Class FQN}::{Group name}`。组名应该短并且用大写字母书写。
使用下划线作为单词分隔符,即 `\Yiisoft\Http\Status::STATUSES`。例如:

```php
expectedReturnValues(
  \Psr\Http\Message\RequestInterface::getMethod(),
  argumentsSet('\Yiisoft\Http\Method::METHODS'),
);

registerArgumentsSet(
  '\Yiisoft\Http\Method::METHODS',
  \Yiisoft\Http\Method::GET,
  \Yiisoft\Http\Method::POST,
  \Yiisoft\Http\Method::PUT,
  \Yiisoft\Http\Method::DELETE,
  \Yiisoft\Http\Method::PATCH,
  \Yiisoft\Http\Method::HEAD,
  \Yiisoft\Http\Method::OPTIONS,
);
```

## PhpStorm 属性

[PhpStorm 属性](https://github.com/JetBrains/phpstorm-attributes) 可以在代码中使用,但包 `jetbrains/phpstorm-attributes` 必须作为开发依赖项添加:

```shell
composer require --dev jetbrains/phpstorm-attributes
```

### 与 ComposerRequireChecker 一起使用

当在同一个包中也使用 [ComposerRequireChecker](https://github.com/maglnet/ComposerRequireChecker) 时,
将涉及的属性的类名添加到配置中的白名单。例如:

```json
{
    "symbol-whitelist": [
        "JetBrains\\PhpStorm\\ExpectedValues",
        "JetBrains\\PhpStorm\\Pure"
    ]
}
```
