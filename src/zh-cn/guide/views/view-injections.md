# 视图注入

视图注入旨在提供一种标准化的方法来将参数传递给应用程序中视图的公共层。它允许开发者管理将在各种视图中可用的数据，确保代码的灵活性和可重用性。

如果您需要 `yiisoft/yii-view-renderer` 包，可以使用视图注入：


```sh
composer require yiisoft/yii-view-renderer
```

## 配置

在配置文件 `params.php` 中：


```php
...
'yiisoft/yii-view' => [
        'injections' => [
            Reference::to(ContentViewInjection::class),
            Reference::to(CsrfViewInjection::class),
            Reference::to(LayoutViewInjection::class),
        ],
    ],
```

## 创建新注入

首先定义一个实现 `Yiisoft\Yii\View\Renderer\CommonParametersInjectionInterface` 的类。此类将负责提供您想要注入到视图模板和布局中的参数。

```php
class MyCustomParametersInjection implements Yiisoft\Yii\View\Renderer\CommonParametersInjectionInterface
{
    // 类属性和方法将放在这里

    public function __construct(UserService $userService)
    {
        $this->userService = $userService;
    }

    public function getCommonParameters(): array
    {
        return [
            'siteName' => 'My Awesome Site',
            'currentYear' => date('Y'),
            'user' => $this->userService->getCurrentUser(),
        ];
    }
}
```

将您的新注入添加到 `params.php`：

```php
'yiisoft/yii-view' => [
        'injections' => [
            ...,
            Reference::to(MyCustomParametersInjection::class),
        ],
    ],
```

## 为不同的布局使用单独的注入

如果您的应用程序有多个布局，您可以为每个布局创建单独的参数注入。这种方法允许您根据每个布局的特定需求定制注入到其中的参数，从而提高应用程序的灵活性和可维护性。

为特定布局创建自定义 ViewInjection：

```php
readonly final class CartViewInjection implements CommonParametersInjectionInterface
{
    public function __construct(private Cart $cart)
    {
    }

    public function getCommonParameters(): array
    {
        return [
            'cart' => $this->cart,
        ];
    }
}
```

在特定的布局名称下将您的新注入添加到 `params.php`。在以下示例中，它是 `@layout/cart`：

```php
'yiisoft/yii-view' => [
        'injections' => [
            ...,
            Reference::to(MyCustomParametersInjection::class),
            DynamicReference::to(static function (ContainerInterface $container) {
                $cart = $container
                    ->get(Cart::class);

                return new LayoutSpecificInjections(
                    '@layout/cart', // 注入的布局名称

                    new CartViewInjection($cart)
                );
            }),
        ],
    ],
```
