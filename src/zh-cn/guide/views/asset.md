# 资源 (Assets)

资源管理对于现代 Web 应用程序至关重要。资源包括 CSS 样式表、JavaScript 文件、图像、字体和其他静态资源。Yii3 通过 `yiisoft/assets` 包提供了一个全面的资源管理系统，用于处理这些资源的依赖关系、优化和部署。

## 安装

资源管理功能由 `yiisoft/assets` 包提供：

```bash
composer require yiisoft/assets
```

此包默认包含在 `yiisoft/app` 应用程序模板中。

## 基本概念

### 资源包 (Asset Bundles)

资源包是逻辑分组在一起的相关资源文件（CSS、JavaScript、图像）的集合。资源包可以依赖于其他包，从而实现适当的依赖关系管理。

### 资源管理器 (Asset Manager)

资源管理器负责：
- 解析资源包依赖关系
- 将资源从受保护目录发布到可访问 Web 的位置
- 合并和压缩资源（配置后）
- 为资源生成正确的 URL

## 创建资源包

### 基本资源包

这是一个简单的资源包定义：

```php
<?php

declare(strict_types=1);

namespace App\Asset;

use Yiisoft\Assets\AssetBundle;

final class MainAsset extends AssetBundle
{
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    public array $css = [
        'css/main.css',
        'css/responsive.css',
    ];

    public array $js = [
        'js/main.js',
        'js/utils.js',
    ];

    public array $depends = [
        BootstrapAsset::class,
        JqueryAsset::class,
    ];
}
```

### 资源包属性

**路径配置：**
- `$basePath` - 资源文件所在的物理路径
- `$baseUrl` - 资源的可访问 Web URL 路径
- `$sourcePath` - 需要发布的资源的源目录

**资源文件：**
- `$css` - CSS 文件数组
- `$js` - JavaScript 文件数组

**依赖关系：**
- `$depends` - 此资源包依赖的其他资源包数组

**选项：**
- `$jsOptions` - JavaScript 标签的 HTML 属性
- `$cssOptions` - CSS 链接标签的 HTML 属性

### 高级资源包

```php
<?php

declare(strict_types=1);

namespace App\Asset;

use Yiisoft\Assets\AssetBundle;

final class AdminAsset extends AssetBundle
{
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    // 带媒体查询的 CSS 文件
    public array $css = [
        'css/admin/main.css',
        ['css/admin/print.css', 'media' => 'print'],
        ['css/admin/mobile.css', 'media' => 'screen and (max-width: 768px)'],
    ];

    // 带属性的 JavaScript 文件
    public array $js = [
        'js/admin/core.js',
        ['js/admin/charts.js', 'defer' => true],
        ['js/admin/analytics.js', 'async' => true],
    ];

    // 此包中所有 JS 文件的全局选项
    public array $jsOptions = [
        'data-admin' => 'true',
    ];

    // 此包中所有 CSS 文件的全局选项
    public array $cssOptions = [
        'data-bundle' => 'admin',
    ];

    public array $depends = [
        MainAsset::class,
        ChartJsAsset::class,
    ];
}
```

## 使用资源包

### 在控制器中

在控制器或视图中注册资源包：

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Asset\MainAsset;
use Psr\Http\Message\ResponseInterface;
use Yiisoft\Assets\AssetManager;
use Yiisoft\Yii\View\Renderer\ViewRenderer;

final class SiteController
{
    public function __construct(
        private ViewRenderer $viewRenderer,
        private AssetManager $assetManager,
    ) {}

    public function index(): ResponseInterface
    {
        // 注册资源包
        $this->assetManager->register(MainAsset::class);

        return $this->viewRenderer->render('index', [
            'title' => 'Home Page',
        ]);
    }

    public function admin(): ResponseInterface
    {
        // 注册多个资源包
        $this->assetManager->register([
            MainAsset::class,
            AdminAsset::class,
        ]);

        return $this->viewRenderer->render('admin/dashboard');
    }
}
```

### 在视图中

您也可以直接在视图中注册资源：

```php
<?php

declare(strict_types=1);

use App\Asset\ProductAsset;
use Yiisoft\Assets\AssetManager;

/**
 * @var \Yiisoft\View\WebView $this
 * @var AssetManager $assetManager
 * @var array $product
 */

// 注册产品特定资源
$assetManager->register(ProductAsset::class);
?>

<div class="product-page">
    <h1><?= Html::encode($product['name']) ?></h1>
    <!-- 产品内容 -->
</div>
```

### 与 WebView 集成

推荐的方法是与 WebView 集成以自动渲染资源：

```php
<?php

declare(strict_types=1);

use App\Asset\MainAsset;

/**
 * @var \Yiisoft\View\WebView $this
 * @var \Yiisoft\Assets\AssetManager $assetManager
 */

// 注册资源
$assetManager->register(MainAsset::class);

// 将所有已注册的资源添加到视图
$this->addCssFiles($assetManager->getCssFiles());
$this->addCssStrings($assetManager->getCssStrings());
$this->addJsFiles($assetManager->getJsFiles());
$this->addJsStrings($assetManager->getJsStrings());
$this->addJsVars($assetManager->getJsVars());
?>

<div class="page-content">
    <!-- 您的页面内容 -->
</div>
```

## 资源发布

### 源路径发布

当资源位于不可访问 Web 的目录（如 vendor 包）中时，需要发布它们：

```php
<?php

declare(strict_types=1);

namespace App\Asset;

use Yiisoft\Assets\AssetBundle;

final class VendorAsset extends AssetBundle
{
    // 源目录（不可通过 Web 访问）
    public string $sourcePath = '@vendor/company/package/assets';

    // 将被发布到可访问 Web 的目录
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    public array $css = [
        'styles.css',
    ];

    public array $js = [
        'script.js',
    ];
}
```

### 自定义发布

您也可以手动发布目录：

```php
/**
 * @var \Yiisoft\Assets\AssetManager $assetManager
 */

// 发布目录
$publishedPath = $assetManager->publish('@vendor/company/package/assets');

// 获取已发布的 URL
$publishedUrl = $assetManager->getPublishedUrl('@vendor/company/package/assets');
```

## 第三方库资源

### jQuery 资源包

```php
<?php

declare(strict_types=1);

namespace App\Asset;

use Yiisoft\Assets\AssetBundle;

final class JqueryAsset extends AssetBundle
{
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    public array $js = [
        'js/jquery-3.6.0.min.js',
    ];

    // 或使用 CDN
    public array $jsOptions = [
        'integrity' => 'sha256-...',
        'crossorigin' => 'anonymous',
    ];
}
```

### Bootstrap 资源包

```php
<?php

declare(strict_types=1);

namespace App\Asset;

use Yiisoft\Assets\AssetBundle;

final class BootstrapAsset extends AssetBundle
{
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    public array $css = [
        'css/bootstrap.min.css',
    ];

    public array $js = [
        'js/bootstrap.bundle.min.js',
    ];

    public array $depends = [
        JqueryAsset::class, // Bootstrap 需要 jQuery
    ];
}
```

### CDN 资源

对于 CDN 托管的资源，可以指定完整 URL：

```php
<?php

declare(strict_types=1);

namespace App\Asset;

use Yiisoft\Assets\AssetBundle;

final class CdnAsset extends AssetBundle
{
    public array $css = [
        'https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css',
    ];

    public array $js = [
        'https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js',
    ];

    public array $jsOptions = [
        'integrity' => 'sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p',
        'crossorigin' => 'anonymous',
    ];
}
```

## 资源配置

### 应用程序配置

在应用程序配置中配置资源管理：

**config/web/params.php**
```php
return [
    'yiisoft/assets' => [
        // 已发布资源的基本路径
        'basePath' => '@webroot/assets',

        // 资源的基本 URL
        'baseUrl' => '@web/assets',

        // 资源转换器配置
        'converter' => [
            'commands' => [
                'scss' => ['sass', '{from}', '{to}', '--style=compressed'],
                'ts' => ['tsc', '{from}', '--outFile', '{to}'],
            ],
        ],
    ],
];
```

### 环境特定资源

为不同环境配置不同的资源：

```php
<?php

declare(strict_types=1);

namespace App\Asset;

use Yiisoft\Assets\AssetBundle;

final class MainAsset extends AssetBundle
{
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    public array $css = [];
    public array $js = [];

    public function __construct()
    {
        if (YII_ENV_DEV) {
            // 开发环境资源（未压缩）
            $this->css = ['css/main.css', 'css/debug.css'];
            $this->js = ['js/main.js', 'js/debug.js'];
        } else {
            // 生产环境资源（已压缩）
            $this->css = ['css/main.min.css'];
            $this->js = ['js/main.min.js'];
        }
    }
}
```

## 资源优化

### 资源合并

将多个 CSS 或 JavaScript 文件合并为单个文件：

```php
/**
 * @var \Yiisoft\Assets\AssetManager $assetManager
 */

// 启用资源合并
$assetManager->setCombine(true);

// 设置合并选项
$assetManager->setCombineOptions([
    'css' => true,  // 合并 CSS 文件
    'js' => true,   // 合并 JavaScript 文件
]);
```

### 资源压缩

配置生产环境的资源压缩：

```php
// 在资源管理器配置中
'converter' => [
    'commands' => [
        'css' => ['cleancss', '{from}', '-o', '{to}'],
        'js' => ['uglifyjs', '{from}', '-o', '{to}', '--compress', '--mangle'],
    ],
],
```

## 使用资源转换器

### SCSS/SASS 编译

```php
<?php

declare(strict_types=1);

namespace App\Asset;

use Yiisoft\Assets\AssetBundle;

final class ScssAsset extends AssetBundle
{
    public string $sourcePath = '@app/assets/scss';
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    public array $css = [
        'main.scss', // 将被转换为 main.css
        'components.scss',
    ];
}
```

### TypeScript 编译

```php
<?php

declare(strict_types=1);

namespace App\Asset;

use Yiisoft\Assets\AssetBundle;

final class TypeScriptAsset extends AssetBundle
{
    public string $sourcePath = '@app/assets/ts';
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    public array $js = [
        'main.ts', // 将被转换为 main.js
        'utils.ts',
    ];
}
```

## 实际示例

### 完整的应用程序资源结构

```
assets/
├── css/
│   ├── main.css           # 主应用程序样式
│   ├── admin.css          # 管理特定样式
│   └── mobile.css         # 移动特定样式
├── js/
│   ├── main.js            # 主应用程序 JavaScript
│   ├── admin.js           # 管理功能
│   └── vendor/            # 第三方库
│       ├── jquery.min.js
│       └── bootstrap.min.js
├── images/
│   ├── logo.png
│   └── icons/
└── fonts/
    ├── main.woff2
    └── main.woff
```

### 完整的资源包设置

```php
<?php

declare(strict_types=1);

namespace App\Asset;

use Yiisoft\Assets\AssetBundle;

// 基础资源包
final class AppAsset extends AssetBundle
{
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    public array $css = [
        'css/main.css',
    ];

    public array $js = [
        'js/main.js',
    ];

    public array $depends = [
        JqueryAsset::class,
    ];
}

// 管理特定包
final class AdminAsset extends AssetBundle
{
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    public array $css = [
        'css/admin.css',
    ];

    public array $js = [
        'js/admin.js',
    ];

    public array $depends = [
        AppAsset::class,
        BootstrapAsset::class,
    ];
}

// 移动特定包
final class MobileAsset extends AssetBundle
{
    public string $basePath = '@assets';
    public string $baseUrl = '@assetsUrl';

    public array $css = [
        ['css/mobile.css', 'media' => 'screen and (max-width: 768px)'],
    ];

    public array $depends = [
        AppAsset::class,
    ];
}
```

## 最佳实践

1. **按功能组织**：将相关资源分组到逻辑包中
2. **管理依赖关系**：正确声明包之间的依赖关系
3. **使用有意义的名称**：清晰命名资源包
4. **环境优化**：在生产环境中使用压缩的资源
5. **CDN 考虑**：适当情况下为流行库使用 CDN
6. **资源版本控制**：在文件名中包含版本号以清除缓存
7. **最小化 HTTP 请求**：尽可能合并相关资源
8. **优化文件大小**：压缩和缩小生产环境的资源

## 故障排除

### 常见问题

**找不到资源：**
- 检查资源文件是否存在于指定路径中
- 验证 `$basePath` 和 `$baseUrl` 配置是否正确
- 如果使用 `$sourcePath`，确保资源已发布

**依赖关系未加载：**
- 验证依赖包是否正确注册
- 检查是否存在循环依赖
- 确保依赖包配置正确

**资源未出现在 HTML 中：**
- 确保在布局中调用了 `$this->addCssFiles()` 和 `$this->addJsFiles()`
- 验证布局中调用了 `$this->head()`、`$this->beginBody()` 和 `$this->endBody()`

**权限问题：**
- 检查 Web 服务器是否具有对资源目录的写入权限
- 验证已发布的资源目录是否可访问 Web
