# 事件

事件允许你在不修改现有代码的情况下，使自定义代码在特定执行点执行。
你可以将称为"处理器"的自定义代码附加到事件，以便在触发事件时，
处理器自动执行。

例如，当用户注册时，你需要发送欢迎电子邮件。你可以直接在
`SignupService` 中执行此操作，但是，当你还需要调整用户头像图像的大小时，你将
不得不再次更改 `SignupService` 代码。换句话说，`SignupService` 将与发送欢迎电子邮件的代码和调整头像图像大小的代码耦合。

为了避免这种情况，你可以不显式告诉注册后做什么，而是引发 `UserSignedUp` 事件，
然后完成注册过程。发送电子邮件的代码和调整头像图像大小的代码将附加到事件，
因此将被执行。如果你需要在注册时做更多事情，你将能够附加额外的事件
处理器而无需修改 `SignupService`。

为了引发事件并将处理器附加到这些事件，Yii 有一个称为事件调度器的特殊服务。
它可从 [yiisoft/event-dispatcher 包](https://github.com/yiisoft/event-dispatcher)获得。

## 事件处理器 <span id="event-handlers"></span>

事件处理器是 [PHP 可调用对象](https://www.php.net/manual/en/language.types.callable.php)，当它附加到的事件被触发时执行。

事件处理器的签名是：

```php
function (EventClass $event) {
    // 处理它
}
```

## 附加事件处理器 <span id="attaching-event-handlers"></span>

你可以像以下那样将处理器附加到事件：

```php
use Yiisoft\EventDispatcher\Provider\Provider;

final readonly class WelcomeEmailSender
{
    public function __construct(Provider $provider)
    {
        $provider->attach([$this, 'handleUserSignup']);
    }

    public function handleUserSignup(UserSignedUp $event)
    {
        // 处理它
    }
}
```

`attach()` 方法接受一个回调。根据此回调参数的类型，确定事件类型。

## 事件处理器顺序

你可以将一个或多个处理器附加到单个事件。当事件被触发时，附加的处理器
将按它们附加到事件的顺序调用。如果事件实现了
`Psr\EventDispatcher\StoppableEventInterface`，事件处理器可以在 `isPropagationStopped()` 返回 `true` 时停止执行其后的其余处理器。

通常，最好不要依赖事件处理器的顺序。

## 引发事件 <span id="raising-events"></span>

事件像以下那样引发：

```php
use Psr\EventDispatcher\EventDispatcherInterface;

final readonly class SignupService
{
    public function __construct(
        private EventDispatcherInterface $eventDispatcher
    )
    {
    }

    public function signup(SignupForm $form)
    {
        // 处理注册

        $event = new UserSignedUp($form);
        $this->eventDispatcher->dispatch($event);
    }
}
```

首先，你创建一个事件，为它提供对处理器有用的数据。然后你调度该事件。

事件类本身可能如下所示：

```php
final readonly class UserSignedUp
{
    public function __construct(
        public SignupForm $form
    )
    {
    }
}
```

## 事件层次结构

事件故意没有任何名称或通配符匹配。事件类名称和类/接口层次结构
和组合可用于实现极大的灵活性：

```php
interface DocumentEvent
{
}

final readonly class BeforeDocumentProcessed implements DocumentEvent
{
}

final readonly class AfterDocumentProcessed implements DocumentEvent
{
}
```

使用接口，你可以监听所有与文档相关的事件：


```php
$provider->attach(function (DocumentEvent $event) {
    // 在这里记录事件
});
```

## 分离事件处理器 <span id="detaching-event-handlers"></span>

要从事件分离处理器，你可以调用 `detach()` 方法：


```php
$provider->detach(DocmentEvent::class);
```

## 配置应用程序事件

你通常通过应用程序配置分配事件处理器。有关详细信息，请参阅["配置"](configuration.md)。
