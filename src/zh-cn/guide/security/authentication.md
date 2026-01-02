# 认证

认证是验证用户身份的过程。它通常使用一个标识符（例如用户名或电子邮件地址）和一个秘密令牌（如密码或访问令牌）来判断用户是否是他所声称的那个人。认证是登录功能的基础。

Yii 提供了一个认证框架，该框架连接各种组件以支持登录。要使用此框架，您主要需要完成以下工作：

* 配置 `Yiisoft\Yii\Web\User\User` 服务；
* 创建一个实现 `\Yiisoft\Auth\IdentityInterface` 接口的类；
* 创建一个实现 `\Yiisoft\Auth\IdentityRepositoryInterface` 接口的类；

## 配置 `Yiisoft\Yii\Web\User\User` <span id="configuring-user"></span>

`Yiisoft\Yii\Web\User\User` 应用程序服务管理用户认证状态。它依赖于 `Yiisoft\Auth\IdentityRepositoryInterface`，该接口应返回一个包含实际认证逻辑的 `\Yiisoft\Auth\IdentityInterface` 实例。

```php
use Yiisoft\Session\Session;
use Yiisoft\Session\SessionInterface;
use Yiisoft\Auth\IdentityRepositoryInterface;
use Psr\Container\ContainerInterface;

return [
    // ...

    SessionInterface::class => [
        'class' => Session::class,
        '__construct()' => [
            $params['session']['options'] ?? [],
            $params['session']['handler'] ?? null,
        ],
    ],


    // User:
    IdentityRepositoryInterface::class => static function (ContainerInterface $container) {
        // 可以使用任何实现来替代基于 Cycle 的仓库
        return $container->get(\Cycle\ORM\ORMInterface::class)->getRepository(\App\Entity\User::class);
    },
];
```

## 实现 `\Yiisoft\Auth\IdentityInterface` <span id="implementing-identity"></span>

身份类必须实现 `\Yiisoft\Auth\IdentityInterface`，该接口只有一个方法：

* [[yii\web\IdentityInterface::getId()|getId()]]：它返回由此身份实例表示的用户的 ID。

在以下示例中，身份类被实现为一个纯 PHP 对象。

```php
<?php

namespace App\User;

use Yiisoft\Auth\IdentityInterface;

final readonly class Identity implements IdentityInterface
{
    public function __construct(
        private string $id
    ) {
    }

    public function getId(): string
    {
        return $this->id;
    }
}
```

## 实现 `\Yiisoft\Auth\IdentityRepositoryInterface` <span id="implementing-identity-repository"></span>

身份仓库类必须实现 `\Yiisoft\Auth\IdentityRepositoryInterface`，该接口具有以下方法：

* `findIdentity(string $id): ?IdentityInterface`：它使用指定的 ID 查找身份类的实例。当您需要通过会话保持登录状态时，将使用此方法。
* `findIdentityByToken(string $token, string $type): ?IdentityInterface`：它使用指定的访问令牌查找身份类的实例。当您需要通过单个秘密令牌认证用户时（例如，在无状态的 REST API 中），将使用此方法。

一个简单的实现可能如下所示：

```php
namespace App\User;

use App\User\Identity;
use \Yiisoft\Auth\IdentityInterface;
use \Yiisoft\Auth\IdentityRepositoryInterface;

final readonly class IdentityRepository implements IdentityRepositoryInterface
{
    private const USERS = [
      [
        'id' => 1,
        'token' => '12345'
      ],
      [
        'id' => 42,
        'token' => '54321'
      ],
    ];

    public function findIdentity(string $id) : ?IdentityInterface
    {
        foreach (self::USERS as $user) {
            if ((string)$user['id'] === $id) {
                return new Identity($id);
            }
        }

        return null;
    }

    public function findIdentityByToken(string $token, string $type) : ?IdentityInterface
    {
        foreach (self::USERS as $user) {
             if ($user['token'] === $token) {
                 return new Identity((string)$user['id']);
             }
         }

         return null;
    }
}
```

## 使用 `\Yiisoft\User\User` <span id="using-user"></span>

您可以使用 `\Yiisoft\User\User` 服务来获取当前用户身份。与任何服务一样，它可以在操作处理程序构造函数或方法中自动注入：

```php
use \Psr\Http\Message\ServerRequestInterface;
use \Yiisoft\User\User;

final readonly class SiteController
{
    public function actionIndex(ServerRequestInterface $request, User $user)
    {
        if ($user->isGuest()) {
            // 用户是访客
        } else {
            $identity = $user->getIdentity();
            // 根据身份执行某些操作
        }
    }
}
```

`isGuest()` 判断用户是否已登录。`getIdentity()` 返回身份实例。

要登录用户，您可以使用以下代码：

```php
$identity = $identityRepository->findByEmail($email);

/* @var $user \Yiisoft\User\User */
$user->login($identity);
```

`login()` 方法将身份设置到 User 服务。它将身份存储到会话中，以便维护用户认证状态。

要登出用户，请调用

```php
/* @var $user \Yiisoft\User\User */
$user->logout();
```

## 认证事件 <span id="auth-events"></span>

用户服务在登录和登出过程中会引发几个事件。

* `\Yiisoft\User\Event\BeforeLogin`：在 `login()` 开始时引发。如果事件处理程序在事件对象上调用 `invalidate()`，登录过程将被取消。
* `\Yiisoft\User\Event\AfterLogin`：在成功登录后引发。
* `\Yiisoft\User\Event\BeforeLogout`：在 `logout()` 开始时引发。如果事件处理程序在事件对象上调用 `invalidate()`，登出过程将被取消。
* `\Yiisoft\User\Event\AfterLogout`：在成功登出后引发。

您可以响应这些事件以实现登录审计、在线用户统计等功能。例如，在 `\Yiisoft\User\Event\AfterLogin` 的处理程序中，您可以在 `user` 数据库表中记录登录时间和 IP 地址。
