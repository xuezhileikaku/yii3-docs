# 授权

授权是验证用户是否有足够权限执行某项操作的过程。

## 检查权限 <span id="checking-for-permission"></span>

您可以使用 `\Yiisoft\User\User` 服务来检查用户是否具有特定权限：

```php
namespace App\Blog\Post;

use Yiisoft\Router\CurrentRoute;
use Yiisoft\User\User;

final readonly class PostController
{
    public function actionEdit(CurrentRoute $route, User $user, PostRepository $postRepository)
    {
        $postId = $route->getArgument('id');
        if ($postId === null) {
            // 响应 404
        }

        $post = $postRepository->findByPK($postId);
        if ($post === null) {
            // 响应 404
        }

        if (!$this->canEditPost($user, $post)) {
            // 响应 403
        }

        // 继续编辑文章
    }

    private function canEditPost(User $user, Post $post): bool
    {
        return $post->getAuthorId() === $user->getId() || $user->can('updatePost');
    }
}
```

在幕后，`Yiisoft\Yii\Web\User\User::can()` 方法调用 `\Yiisoft\Access\AccessCheckerInterface::userHasPermission()`，因此您应该在依赖容器中提供一个实现才能使其工作。

## 基于角色的访问控制 (RBAC) <span id="rbac"></span>

基于角色的访问控制（RBAC）提供了一种简单而强大的集中式访问控制。请参阅 [Wikipedia](https://en.wikipedia.org/wiki/Role-based_access_control) 以了解有关 RBAC 与其他更传统的访问控制方案比较的详细信息。

Yii 实现了一个通用的分层 RBAC，遵循 [NIST RBAC 模型](https://csrc.nist.gov/CSRC/media/Publications/conference-paper/2000/07/26/the-nist-model-for-role-based-access-control-towards-a-unified-/documents/sandhu-ferraiolo-kuhn-00.pdf)。

使用 RBAC 涉及两部分工作。第一部分是构建 RBAC 授权数据，第二部分是在必要的地方使用授权数据执行访问检查。由于 RBAC 实现了 `\Yiisoft\Access\AccessCheckerInterface`，因此使用它类似于使用任何其他访问检查器实现。

为了便于接下来的描述，首先介绍一些基本的 RBAC 概念。

### 基本概念 <span id="basic-concepts"></span>

角色代表一组*权限*（例如，创建文章、更新文章）。您可以为一个或多个用户分配角色。要检查用户是否具有指定权限，您可以检查用户是否具有具有该权限的角色。

与每个角色或权限关联的，可能有一个*规则*。规则表示访问检查器将执行的一段代码，以决定相应的角色或权限是否适用于当前用户。例如，"更新文章"权限可能有一个规则来检查当前用户是否是文章创建者。在访问检查期间，如果用户不是文章创建者，则没有"更新文章"权限。

角色和权限都在层次结构中。特别是，角色可能由其他角色或权限组成。权限可能由其他权限组成。Yii 实现了一个*偏序*层次结构，其中包括更特殊的*树*层次结构。虽然角色可以包含权限，但反过来并非如此。

### 配置 RBAC <span id="configuring-rbac"></span>

RBAC 通过 `yiisoft/rbac` 包提供，因此您需要安装它：

```
composer require yiisoft/rbac
```

在开始定义授权数据并执行访问检查之前，您需要在依赖容器中配置 `\Yiisoft\Access\AccessCheckerInterface`：

```php
use \Psr\Container\ContainerInterface;
use Yiisoft\Rbac\Manager\PhpManager;
use Yiisoft\Rbac\RuleFactory\ClassNameRuleFactory;

return [
    \Yiisoft\Access\AccessCheckerInterface::class => static function (ContainerInterface $container) {
        $aliases = $container->get(\Yiisoft\Aliases\Aliases::class);
        return new PhpManager(new ClassNameRuleFactory(), $aliases->get('@rbac'));
    }
];
```

`\Yiisoft\Rbac\Manager\PhpManager` 使用 PHP 脚本文件来存储授权数据。这些文件位于 `@rbac` 别名下。如果您想在线更改权限层次结构，请确保目录及其中的所有文件可由 Web 服务器进程写入。

### 构建授权数据 <span id="generating-rbac-data"></span>

构建授权数据涉及以下任务：

- 定义角色和权限；
- 建立角色和权限之间的关系；
- 定义规则；
- 将规则与角色和权限关联；
- 为用户分配角色。

根据授权灵活性要求，您可以以不同方式执行这些任务。如果只有开发人员更改您的权限层次结构，您可以使用迁移或控制台命令。迁移的优势是您可以与其他迁移一起执行它。控制台命令的优势是您可以在代码中很好地了解层次结构，而无需阅读许多迁移。

无论哪种方式，最终您都会得到以下 RBAC 层次结构：

![简单的 RBAC 层次结构](/images/guide/security/rbac-hierarchy-1.svg "简单的 RBAC 层次结构")

如果您想动态构建权限层次结构，则需要 UI 或控制台命令。用于构建层次结构的 API 本身不会有任何不同。

### 使用控制台命令

如果您的权限层次结构根本不变，并且您有固定数量的用户，您可以创建一个[控制台命令](../tutorial/console-applications.md)，该命令将通过 `\Yiisoft\Rbac\ManagerInterface` 提供的 API 一次初始化授权数据：

```php
<?php
namespace App\Command;

use Symfony\Component\Console\Attribute\AsCommand;use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Yiisoft\Rbac\ManagerInterface;
use Yiisoft\Rbac\Permission;
use Yiisoft\Rbac\Role;
use Yiisoft\Yii\Console\ExitCode;

#[AsCommand(
    name: 'rbac:init',
    description: '构建 RBAC 层次结构',
)]
final readonly class RbacCommand extends Command
{
    public function __construct(
        private ManagerInterface $manager
    ) {
    }

    protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $auth = $this->manager;

        $auth->removeAll();

        $createPost = (new Permission('createPost'))->withDescription('创建文章');
        $auth->add($createPost);

        $updatePost = (new Permission('updatePost'))->withDescription('更新文章');
        $auth->add($updatePost);

        // 添加 "author" 角色并为此角色授予 "createPost" 权限
        $author = new Role('author');
        $auth->add($author);
        $auth->addChild($author, $createPost);

        // 添加 "admin" 角色并为此角色授予 "updatePost" 权限
        // 以及 "author" 角色的权限
        $admin = new Role('admin');
        $auth->add($admin);
        $auth->addChild($admin, $updatePost);
        $auth->addChild($admin, $author);

        // 为用户分配角色。1 和 2 是 IdentityInterface::getId() 返回的 ID
        // 通常在您的 User 模型中实现。
        $auth->assign($author, 2);
        $auth->assign($admin, 1);

        return ExitCode::OK;
    }
}
```

您可以通过以下方式从控制台执行上述命令：

```
./yii rbac:init
```

> 如果您不想硬编码哪些用户具有特定角色，请不要将 `->assign()` 调用放入命令中。相反，创建 UI 或控制台命令来管理分配。

#### 使用迁移

**TODO**：在实现迁移时完成。

您可以使用[迁移](../databases/db-migrations.md)来通过 `\Yiisoft\Rbac\ManagerInterface` 提供的 API 初始化和更改层次结构。

使用 `./yii migrate:create init_rbac` 创建新迁移，然后实现创建层次结构：

```php
<?php
use yii\db\Migration;

use Yiisoft\Rbac\ManagerInterface;
use Yiisoft\Rbac\Permission;
use Yiisoft\Rbac\Role;

class m170124_084304_init_rbac extends Migration
{
    public function up()
    {
        $auth = /* 获取 auth */;

        $auth->removeAll();

        $createPost = (new Permission('createPost'))->withDescription('创建文章');
        $auth->add($createPost);

        $updatePost = (new Permission('updatePost'))->withDescription('更新文章');
        $auth->add($updatePost);

        // 添加 "author" 角色并为此角色授予 "createPost" 权限
        $author = new Role('author');
        $auth->add($author);
        $auth->addChild($author, $createPost);

        // 添加 "admin" 角色并为此角色授予 "updatePost" 权限
        // 以及 "author" 角色的权限
        $admin = new Role('admin');
        $auth->add($admin);
        $auth->addChild($admin, $updatePost);
        $auth->addChild($admin, $author);

        // 为用户分配角色。1 和 2 是 IdentityInterface::getId() 返回的 ID
        // 通常在您的 User 模型中实现。
        $auth->assign($author, 2);
        $auth->assign($admin, 1);
    }

    public function down()
    {
        $auth = /* 获取 auth */;

        $auth->removeAll();
    }
}
```

> 如果您不想硬编码哪些用户具有特定角色，请不要将 `->assign()` 调用放入迁移中。相反，创建 UI 或控制台命令来管理分配。

您可以使用 `./yii migrate` 应用迁移。

## 为用户分配角色

TODO：在演示/模板中实现注册时更新。

作者可以创建文章，管理员可以更新文章并执行作者可以做的所有事情。

如果您的应用程序允许用户注册，您需要立即为这些新用户分配角色。例如，为了让所有注册用户在您的高级项目模板中成为作者，您需要按如下方式更改 `frontend\models\SignupForm::signup()`：

```php
public function signup()
{
    if ($this->validate()) {
        $user = new User();
        $user->username = $this->username;
        $user->email = $this->email;
        $user->setPassword($this->password);
        $user->generateAuthKey();
        $user->save(false);

        // 添加了以下三行：
        $auth = \Yii::$app->authManager;
        $authorRole = $auth->getRole('author');
        $auth->assign($authorRole, $user->getId());

        return $user;
    }

    return null;
}
```

对于需要复杂访问控制且动态更新授权数据的应用程序（如管理面板），您可能需要使用 `authManager` 提供的 API 开发特殊的用户界面。


### 使用规则 <span id="using-rules"></span>

如前所述，规则为角色和权限添加了额外的约束。规则是一个从 `\Yiisoft\Rbac\Rule` 扩展的类。它必须实现 `execute()` 方法。在您之前创建的层次结构中，作者无法编辑自己的文章。让我们修复它。首先，您需要一个规则来验证用户是文章作者：

```php
namespace App\User\Rbac;

use Yiisoft\Rbac\Item;
use \Yiisoft\Rbac\Rule;

/**
 * 检查 authorID 是否与通过参数传递的用户匹配。
 */
final readonly class AuthorRule extends Rule
{
    private const NAME = 'isAuthor';

    public function __construct() {
        parent::__construct(self::NAME);
    }

    public function execute(string $userId, Item $item, array $parameters = []): bool
    {
        return isset($params['post']) ? $params['post']->getAuthorId() == $userId : false;
    }
}
```

该规则检查用户是否创建了 `post`。在您之前使用的命令中创建一个特殊权限 `updateOwnPost`：

```php
/** @var \Yiisoft\Rbac\ManagerInterface $auth */

// 添加规则
$rule = new AuthorRule();
$auth->add($rule);

// 添加 "updateOwnPost" 权限并将规则与其关联。
$updateOwnPost = (new \Yiisoft\Rbac\Permission('updateOwnPost'))
    ->withDescription('更新自己的文章')
    ->withRuleName($rule->getName());
$auth->add($updateOwnPost);

// "updateOwnPost" 将从 "updatePost" 使用
$auth->addChild($updateOwnPost, $updatePost);

// 允许 "author" 更新自己的文章
$auth->addChild($author, $updateOwnPost);
```

现在您已经得到了以下层次结构：

![带有规则的 RBAC 层次结构](/images/guide/security/rbac-hierarchy-2.svg "带有规则的 RBAC 层次结构")


### 访问检查 <span id="access-check"></span>

检查的方式与在本指南第一部分中的方式类似：

```php
namespace App\Blog\Post;

use Psr\Http\Message\ServerRequestInterface;
use Yiisoft\User\User;

final readonly class PostController
{
    public function actionEdit(ServerRequestInterface $request, User $user, PostRepository $postRepository)
    {
        $postId = $request->getAttribute('id');
        if ($postId === null) {
            // 响应 404
        }

        $post = $postRepository->findByPK($postId);
        if ($post === null) {
            // 响应 404
        }

        if (!$this->canEditPost($user, $post)) {
            // 响应 403
        }

        // 继续编辑文章
    }

    private function canEditPost(User $user, Post $post): bool
    {
        return $user->can('updatePost', ['post' => $post]);
    }
}
```

区别在于现在检查用户自己的文章是 RBAC 的一部分。

如果当前用户是 ID=1 的 Jane，您将从 `createPost` 开始并尝试到达 `Jane`：

![访问检查](/images/guide/security/rbac-access-check-1.svg "访问检查")

要检查用户是否可以更新文章，您需要传递前面描述的 `AuthorRule` 所需的额外参数：

```php
if ($user->can('updatePost', ['post' => $post])) {
    // 更新文章
}
```

如果当前用户是 John，会发生以下情况：

![访问检查](/images/guide/security/rbac-access-check-2.svg "访问检查")

您从 `updatePost` 开始并经过 `updateOwnPost`。要通过访问检查，`AuthorRule` 应该从其 `execute()` 方法返回 `true`。该方法从 `can()` 方法调用接收其 `$params`，因此值为 `['post' => $post]`。如果一切正常，您将到达分配给 John 的 `author`。

在 Jane 的情况下，这要简单一些，因为她是管理员：

![访问检查](/images/guide/security/rbac-access-check-3.svg "访问检查")

## 实现您自己的访问检查器

如果 RBAC 不适合您的需求，您可以在不更改应用程序代码的情况下实现自己的访问检查器：


```php
namespace App\User;

use \Yiisoft\Access\AccessCheckerInterface;

final readonly class AccessChecker implements AccessCheckerInterface
{
    private const PERMISSIONS = [
        [
            1 => ['editPost'],
            42 => ['editPost', 'deletePost'],
        ],
    ];

    public function userHasPermission($userId, string $permissionName, array $parameters = []) : bool
    {
        if (!array_key_exists($userId, self::PERMISSIONS)) {
            return false;
        }

        return in_array($permissionName, self::PERMISSIONS[$userId], true);
    }
}
```
