# 扩展包

可重用的代码可以作为 [Composer 包](https://getcomposer.org/doc/05-repositories.md#package) 发布。
它可以是基础设施库、代表应用程序上下文之一的模块，或者基本上是任何可重用的代码。

## 使用扩展包 <span id="using-packages"></span>

默认情况下，Composer 安装在 [Packagist](https://packagist.org/) 上注册的包——这是最大的
开源 PHP 包存储库。您可以在 Packagist 上查找包。您也可以
[创建自己的存储库](https://getcomposer.org/doc/05-repositories.md#repository) 并配置 Composer
使用它。如果您正在开发只想在项目之间共享的私有包，这很有用。

Composer 安装的包存储在项目的 `vendor` 目录中。
因为 Composer 是依赖管理器，所以当它安装包时，它还将安装其所有依赖包。

> [!WARNING]
> 不应修改应用程序的 `vendor` 目录。

可以使用以下命令安装包：

```
composer install vendor-name/package-name
```

完成后，Composer 会修改 `composer.json` 和 `composer.lock`。前者定义要安装的包
及其版本约束，后者存储实际安装的确切版本的快照。

包中的类将通过[自动加载](../concept/autoloading.md)立即可用。

## 创建扩展包 <span id="creating-packages"></span>


当您觉得需要与他人分享您的优秀代码时，您可以考虑创建一个包。
包可以包含您喜欢的任何代码，例如辅助类、小部件、服务、中间件、整个模块等。

以下是您可以遵循的基本步骤。

1. 为您的包创建一个项目并将其托管在 VCS 存储库中，例如 [GitHub.com](https://github.com)。
   包的开发和维护工作应在此存储库上进行。
2. 在项目的根目录下，根据 Composer 的要求创建一个名为 `composer.json` 的文件。请
   参阅下一小节以获取更多详细信息。
3. 在 Composer 存储库（例如 [Packagist](https://packagist.org/)）中注册您的包，以便
   其他用户可以使用 Composer 查找和安装您的包。


### `composer.json` <span id="composer-json"></span>

每个 Composer 包必须在其根目录中有一个 `composer.json` 文件。该文件包含有关
包的元数据。您可以在 [Composer 手册](https://getcomposer.org/doc/01-basic-usage.md#composer-json-project-setup)中找到有关此文件的完整规范。
以下示例显示了 `yiisoft/yii-widgets` 包的 `composer.json` 文件：

```json
{
    "name": "yiisoft/yii-widgets",
    "type": "library",
    "description": "Yii 小部件集合",
    "keywords": [
        "yii",
        "widgets"
    ],
    "homepage": "https://www.yiiframework.com/",
    "license": "BSD-3-Clause",
    "support": {
        "issues": "https://github.com/yiisoft/yii-widgets/issues?state=open",
        "forum": "https://www.yiiframework.com/forum/",
        "wiki": "https://www.yiiframework.com/wiki/",
        "irc": "ircs://irc.libera.chat:6697/yii",
        "chat": "https://t.me/yii3en",
        "source": "https://github.com/yiisoft/yii-widgets"
    },
    "funding": [
        {
            "type": "opencollective",
            "url": "https://opencollective.com/yiisoft"
        },
        {
            "type": "github",
            "url": "https://github.com/sponsors/yiisoft"
        }
    ],
    "require": {
        "php": "^7.4|^8.0",
        "yiisoft/aliases": "^1.1|^2.0",
        "yiisoft/cache": "^1.0",
        "yiisoft/html": "^2.0",
        "yiisoft/view": "^4.0",
        "yiisoft/widget": "^1.0"
    },
    "require-dev": {
        "phpunit/phpunit": "^9.5",
        "roave/infection-static-analysis-plugin": "^1.16",
        "spatie/phpunit-watcher": "^1.23",
        "vimeo/psalm": "^4.18",
        "yiisoft/psr-dummy-provider": "^1.0",
        "yiisoft/test-support": "^1.3"
    },
    "autoload": {
        "psr-4": {
            "Yiisoft\\Yii\\Widgets\\": "src"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Yiisoft\\Yii\\Widgets\\Tests\\": "tests"
        }
    },
    "extra": {
        "branch-alias": {
            "dev-master": "3.0.x-dev"
        }
    },
    "scripts": {
        "test": "phpunit --testdox --no-interaction",
        "test-watch": "phpunit-watcher watch"
    },
    "config": {
        "sort-packages": true,
        "allow-plugins": {
            "infection/extension-installer": true,
            "composer/package-versions-deprecated": true
        }
    }
}
```


#### 包名称 <span id="package-name"></span>

每个 Composer 包都应有一个包名称，用于在所有包中唯一标识该包。
包名称的格式是 `vendorName/projectName`。例如，在包名称 `yiisoft/queue` 中，
供应商名称和项目名称分别是 `yiisoft` 和 `queue`。

> [!WARNING]
> 不要使用 `yiisoft` 作为您的供应商名称，因为它保留供 Yii 本身使用。

我们建议您对于无法作为通用 PHP 包工作且需要 Yii 应用程序的包，在项目名称前加上 `yii-` 前缀。
这将让用户更容易判断包是否特定于 Yii。


#### 依赖项 <span id="dependencies"></span>

如果您的扩展依赖于其他包，您应该在 `composer.json` 的 `require` 部分中列出它们。
确保为每个依赖包列出适当的版本约束（例如 `^1.0`、`@stable`）。
当您的扩展以稳定版本发布时，请使用稳定的依赖项。

#### 类自动加载 <span id="class-autoloading"></span>

为了使您的类能够自动加载，您应该在 `composer.json` 文件中指定 `autoload` 条目，
如下所示：

```json
{
    // ....

    "autoload": {
        "psr-4": {
            "MyVendorName\\MyPackageName\\": "src"
        }
    }
}
```

您可以列出一个或多个根命名空间及其相应的文件路径。

### 推荐实践 <span id="recommended-practices"></span>

因为包旨在供他人使用，所以在开发期间您经常需要付出额外的努力。
下面，我们介绍在创建高质量扩展时的一些常见和推荐实践。


#### 测试 <span id="testing"></span>

您希望您的包能够完美运行，而不会给他人带来问题。为了实现这一目标，您应该
在向公众发布扩展之前对其进行测试。

建议您创建各种测试用例来覆盖您的扩展代码，而不是依赖手动测试。
每次发布新版本的包之前，您可以运行这些测试用例以确保
一切正常。有关更多详细信息，请参阅[测试](../testing/overview.md)部分。


#### 版本控制 <span id="versioning"></span>

您应该为扩展的每个版本赋予一个版本号（例如 `1.0.1`）。我们建议您在确定应使用什么版本号时遵循
[语义化版本](https://semver.org)实践。


#### 发布 <span id="releasing"></span>

为了让其他人了解您的包，您需要向公众发布它。

如果您是第一次发布包，您应该在 Composer 存储库（例如
[Packagist](https://packagist.org/)）中注册它。
之后，您只需在扩展的 VCS 存储库上创建一个发布标签（例如 `v1.0.1`）
并通知 Composer 存储库有关新发布的信息。人们
然后将能够找到新版本并通过 Composer 存储库安装或更新包。

在发布您的包时，除了代码文件外，您还应该考虑包括以下内容以
帮助其他人了解和使用您的扩展：

* 包根目录中的自述文件：它描述您的扩展的作用以及如何安装和使用它。
  我们建议您以 [Markdown](https://daringfireball.net/projects/markdown/) 格式编写该文件
  并将其命名为 `README.md`。
* 包根目录中的变更日志文件：它列出每个版本中所做的更改。该文件
  可以以 Markdown 格式编写并命名为 `CHANGELOG.md`。
* 包根目录中的升级文件：它提供有关如何从扩展的旧版本升级的
  说明。该文件可以以 Markdown 格式编写并命名为 `UPGRADE.md`。
* 教程、演示、屏幕截图等：如果您的扩展提供许多无法在自述文件中完全涵盖的功能，则需要这些。
* API 文档：您的代码应该有很好的文档记录，以便其他人能够更容易地阅读和理解它。
