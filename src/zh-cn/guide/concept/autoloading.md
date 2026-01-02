# 类自动加载

由于 Yii 使用 [Composer](https://getcomposer.org) 来管理包，它会自动从这些包加载类
而无需显式地 `require` 它们的文件。
当它安装包时，它会生成一个[符合 PSR-4 的自动加载器](https://www.php-fig.org/psr/psr-4/)。
要使用它，在你的 `index.php` 入口点文件中 `require_once` 自动加载器 `/vendor/autoload.php`。

你不仅可以为已安装的包使用自动加载器，也可以为你的应用程序使用，因为它也是一个包。
要加载某个命名空间的类，将以下内容添加到 `composer.json`：

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

其中 `App\\` 是根命名空间，`src/` 是你拥有类的目录。如果需要，你可以添加更多源根目录。
完成后，执行 `composer dump-autoload` 或简单地 `composer du`，来自相应命名空间的类将开始自动加载。

如果你需要特定于开发环境的自动加载，而在使用 `--no-dev` 标志执行 Composer 时不使用，
则将其添加到 `autoload-dev` 部分而不是 `autoload`。

## 参考

- [PSR-4：自动加载器](https://www.php-fig.org/psr/psr-4/)。
- [Composer 关于自动加载的指南](https://getcomposer.org/doc/01-basic-usage.md#autoloading)。
