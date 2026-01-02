# 使用表单

本节继续改进"Say Hello"。不再使用 URL，现在将通过表单向用户询问消息。

通过本教程，你将学习如何：

* 创建表单模型来表示用户通过表单输入的数据。
* 声明规则来验证输入的数据。
* 在视图中构建 HTML 表单。

## 安装表单包

要安装表单包，请在你的应用程序目录中发出以下命令：

```
composer require yiisoft/form-model
```

对于 Docker，应该是：

```
make composer require yiisoft/form-model
```

## 创建表单 <span id="creating-form"></span>

要从用户那里请求的数据将由 `Form` 类表示，如下所示并保存在文件 `/src/App/Web/Echo/Form.php` 中：

```php
<?php

declare(strict_types=1);

namespace App\Web\Echo;

use Yiisoft\FormModel\FormModel;
use Yiisoft\Validator\Label;
use Yiisoft\Validator\Rule\Length;

final class Form extends FormModel
{
    #[Label('The message to be echoed')]
    #[Length(min: 2)]
    public string $message = '';
}
```

在上面的示例中，`Form` 有一个字符串属性 `$message`，其长度应至少为两个字符。还有一个该属性的自定义标签。

## 使用表单 <span id="using-form"></span>

现在你有了一个表单，在来自"[Say Hello](hello.md)"的操作中使用它。

这是你在 `/src/Web/Echo/Action.php` 中最终得到的结果：

```php
<?php

declare(strict_types=1);

namespace App\Web\Echo;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Yiisoft\FormModel\FormHydrator;
use Yiisoft\Yii\View\Renderer\ViewRenderer;

final readonly class Action
{
    public function __construct(
        private ViewRenderer $viewRenderer,
        private FormHydrator $formHydrator,
    ) {}

    public function __invoke(ServerRequestInterface $request): ResponseInterface
    {
        $form = new Form();

        $this->formHydrator->populateFromPostAndValidate($form, $request);

        return $this->viewRenderer->render(__DIR__ . '/template', [
            'form' => $form,
        ]);
    }
}
```

你不再从路由中读取，而是从请求的 POST 数据中填充表单并在 `FormHydrator` 的帮助下验证它。接下来，将表单传递给视图。

为了使表单正常工作，我们需要允许 GET 来呈现表单和 POST 来发送数据。
在 `config/common/routes.php` 中调整你的路由：

```php
<?php

declare(strict_types=1);

use App\Web;
use Yiisoft\Http\Method;
use Yiisoft\Router\Group;
use Yiisoft\Router\Route;

return [
    Group::create()
        ->routes(
            Route::get('/')
                ->action(Web\HomePage\Action::class)
                ->name('home'),
            Route::methods([Method::GET, Method::POST], '/say')
                ->action(Web\Echo\Action::class)
                ->name('echo/say'),
        ),
];
```

## 调整视图

要呈现表单，你需要更改视图 `src/Web/Echo/template.php`：

```php
<?php
use App\Web\Echo\Form;
use Yiisoft\FormModel\Field;
use Yiisoft\Html\Html;
use Yiisoft\Router\UrlGeneratorInterface;
use Yiisoft\Yii\View\Renderer\Csrf;

/**
 * @var Form $form
 * @var string[] $errors
 * @var UrlGeneratorInterface $urlGenerator
 * @var Csrf $csrf
 */

$htmlForm = Html::form()
    ->post($urlGenerator->generate('echo/say'))
    ->csrf($csrf);
?>

<?= $htmlForm->open() ?>
    <?= Field::text($form, 'message')->required() ?>
    <?= Html::submitButton('Say') ?>
<?= $htmlForm->close() ?>

<?php if ($form->isValid()): ?>
    Echo said: <?= Html::encode($form->message) ?>
<?php endif ?>
```

如果表单有效，你将显示一条消息。其余部分初始化并呈现表单。

首先，你使用 POST 类型和 URL 生成器帮助生成的操作 URL 来初始化 `$htmlForm`。你可以在所有视图中以 `$urlGenerator` 的形式访问它。你还需要将 CSRF 令牌传递给表单，由于 `config/common/params.php` 中列出的视图注入，它在每个视图中也以 `$csrf` 的形式可用：

```php
'yiisoft/yii-view-renderer' => [
    'injections' => [
        Reference::to(CsrfViewInjection::class),
    ],
],
```

模板将 CSRF 令牌值呈现为隐藏输入，以确保请求来自表单页面而不是来自其他网站。它将随 POST 表单数据一起提交。省略它将导致 [HTTP 响应代码 422](https://tools.ietf.org/html/rfc4918#section-11.2)。

你使用 `Field::text()` 来输出"message"字段，因此它会处理填充值、转义值、呈现字段标签和验证错误。

现在，如果你提交空消息，你将收到验证错误："The message to be echoed must contain at least 2 characters."

## 试用 <span id="trying-it-out"></span>

要查看它是如何工作的，请使用浏览器访问以下 URL：

```
http://localhost:8080/say
```

你将看到一个页面，其中包含一个表单输入字段和一个指示要输入什么数据的标签。此外，表单还有一个标记为"Say"的"提交"按钮。如果你在未输入任何内容的情况下单击"提交"按钮，你将看到该字段是必填的。如果你输入单个字符，表单会在有问题的输入字段旁边显示错误消息。

![带有验证错误的表单](/images/guide/start/form-error.png)

输入有效消息并单击"提交"按钮后，页面会回显你输入的数据。

![带有成功消息的表单](/images/guide/start/form-success.png)

## 总结 <span id="summary"></span>

在本指南的这一部分中，你学习了如何创建表单模型类来表示用户数据并验证所述数据。

你还学习了如何从用户获取数据以及如何在浏览器中显示数据。这是在开发应用程序时可能花费大量时间的任务，但 Yii 提供了强大的小部件来简化这项任务。

在下一节中，你将学习如何使用数据库，这几乎是每个应用程序都需要的。
