# 邮件发送

Yii 使用 [yiisoft/mailer](https://github.com/yiisoft/mailer) 包简化了电子邮件消息的撰写和发送。该包提供了内容组合功能和发送电子邮件的基本接口。默认情况下，该包包含一个文件邮件程序，将电子邮件内容写入文件而不是发送它们。这在应用程序开发的初始阶段特别有用。

要发送实际的电子邮件，您可以使用 [Symfony Mailer](https://github.com/yiisoft/mailer-symfony) 实现，以下示例中使用了它。

## 配置邮件程序

邮件程序服务允许您创建消息实例、用数据填充它并发送它。通常，您从 DI 容器获取实例作为 `Yiisoft\Mailer\MailerInterface`。

您也可以手动创建实例，如下所示：

```php

use Yiisoft\Mailer\Symfony\Mailer;

/**
 * @var \Symfony\Component\Mailer\Transport\TransportInterface $transport
 */

$mailer = new \Yiisoft\Mailer\Symfony\Mailer(
    $transport,
);
```

`Yiisoft\Mailer\MailerInterface` 提供两个主要方法：

- `send()` - 发送给定的电子邮件消息。
- `sendMultiple()` - 一次发送多条消息。

## 创建消息

### 简单文本消息

要创建带有文本正文的简单消息，请使用 `Yiisoft\Mailer\Message`：

```php
$message = new \Yiisoft\Mailer\Message(
    from: 'from@domain.com',
    to: 'to@domain.com',
    subject: 'Message subject',
    textBody: 'Plain text content'
);
```

### 简单 HTML 消息

```php
$message = new \Yiisoft\Mailer\Message(
    from: 'from@domain.com',
    to: 'to@domain.com',
    subject: 'Message subject',
    htmlBody: '<b>HTML content</b>'
);
```

### 从模板创建 HTML 消息

对于此示例，我们将使用包渲染包 [view](https://github.com/yiisoft/view)。

```php
/**
 * @var \Yiisoft\View\View $view
 */

$content = $view->render('path/to/view.php', [
    'name' => 'name',
    'code' => 'code',
]);

$message = new \Yiisoft\Mailer\Message(
    from: 'from@domain.com',
    to: 'to@domain.com',
    subject: 'Subject',
    htmlBody: $content
);
```

### 使用布局

您还可以从模板消息向布局传递参数：

```php
/**
 * @var \Yiisoft\View\View $view
 * @var array $layoutParameters
 */

$messageBody = $view->render('path/to/view.php', [
    'name' => 'name',
    'code' => 'code',
]);

$layoutParameters['content'] = $messageBody;

$content = $view->render('path/to/layout.php', $layoutParameters);

$message = new \Yiisoft\Mailer\Message(
    from: 'from@domain.com',
    to: 'to@domain.com',
    subject: 'Subject',
    htmlBody: $content
);
```

### 布局示例

您可以将视图渲染结果包装在布局中，类似于 Web 应用程序中布局的工作方式。这对于设置共享内容（如 CSS 样式）很有用：

```php
<?php
/* @var $content string Mail contents as view render result */
?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>
<body>

<?= $content ?>

<footer style="margin-top: 5em">
-- <br>
Mailed by Yii
</footer>

</body>
</html>
```

## 添加更多数据

`Yiisoft\Mailer\MessageInterface` 提供了几种方法来自定义您的消息：

- `withCharset()` - 返回具有指定字符集的新实例。
- `withFrom()` - 返回具有指定发件人电子邮件地址的新实例。
- `withTo()` - 返回具有指定收件人电子邮件地址的新实例。
- `withReplyTo()` - 返回具有指定回复地址的新实例。
- `withCc()` - 返回具有指定抄送地址的新实例。
- `withBcc()` - 返回具有指定密件抄送地址的新实例。
- `withSubject()` - 返回具有指定消息主题的新实例。
- `withDate()` - 返回具有指定发送日期的新实例。
- `withPriority()` - 返回具有指定消息优先级的新实例。
- `withReturnPath()` - 返回具有指定返回路径（退回地址）的新实例。
- `withSender()` - 返回具有指定实际发件人电子邮件地址的新实例。
- `withHtmlBody()` - 返回具有指定消息 HTML 内容的新实例。
- `withTextBody()` - 返回具有指定消息纯文本内容的新实例。
- `withAddedHeader()` - 返回具有指定添加的自定义标头值的新实例。
- `withHeader()` - 返回具有指定自定义标头值的新实例。
- `withHeaders()` - 返回具有指定自定义标头值的新实例。

这些方法是不可变的，意味着它们返回具有更新数据的新消息实例。

注意 `with` 前缀。它表示该方法是不可变的，并返回具有更改数据的新类实例。

### Getter 方法

以下 getter 方法可用于检索消息数据：

- `getCharset()` - 返回此消息的字符集。
- `getFrom()` - 返回消息发件人电子邮件地址。
- `getTo()` - 返回消息收件人电子邮件地址。
- `getReplyTo()` - 返回此消息的回复地址。
- `getCc()` - 返回此消息的抄送地址。
- `getBcc()` - 返回此消息的密件抄送地址。
- `getSubject()` - 返回消息主题。
- `getDate()` - 返回消息发送日期，如果未设置则返回 null。
- `getPriority()` - 返回此消息的优先级。
- `getReturnPath()` - 返回此消息的返回路径（退回地址）。
- `getSender()` - 返回消息实际发件人电子邮件地址。
- `getHtmlBody()` - 返回消息 HTML 正文。
- `getTextBody()` - 返回消息文本正文。
- `getHeader()` - 返回指定标头的所有值。
- `__toString()` - 返回此消息的字符串表示。

## 附加文件

您可以使用 `withAttached()` 方法将文件附加到消息：

```php
use Yiisoft\Mailer\File;

// 从本地文件系统附加文件
$message = $message->withAttached(
    File::fromPath('/path/to/source/file.pdf'),
);

// 即时创建附件
$message = $message->withAttached(
    File::fromContent('Attachment content', 'attach.txt', 'text/plain'),
);
```

## 嵌入图像

您可以使用 `withEmbedded()` 方法将图像嵌入到消息内容中。这在编写带有视图的消息时特别有用：

```php
$logo = 'path/to/logo';
$htmlBody = $this->view->render(
    __DIR__ . 'template.php',
    [
        'content' => $content,
        'logoCid' => $logo->cid(),
    ],
);
return new \Yiisoft\Mailer\Message(
            from: 'from@domain.com',
            to: 'to@domain.com',
            subject: 'Message subject',
            htmlBody: $htmlBody,
            embeddings: $logo
        );
```

在视图或布局模板中，您可以使用其 CID 引用嵌入的图像：

```php
<img src="<?= $logoCid; ?>">
```

## 发送消息

要发送电子邮件消息：

```php
/**
 * @var \Yiisoft\View\View $view
 */

$content = $view->render('path/to/view.php', [
    'name' => 'name',
    'code' => 'code',
]);

$message = new \Yiisoft\Mailer\Message(
    from: 'from@domain.com',
    to: 'to@domain.com',
    subject: 'Subject',
    htmlBody: $content
);

$mailer->send($message);
```

## 发送多条消息

您可以一次发送多条消息：

```php
$messages = [];

foreach ($users as $user) {
    $messages[] = (new \Yiisoft\Mailer\Message())
        // ...
        ->withTo($user->email);
}

$result = $mailer->sendMultiple($messages);
```

`sendMultiple()` 方法返回一个 `Yiisoft\Mailer\SendResults` 对象，其中包含成功发送和失败的消息数组。

## 实现您自己的邮件驱动程序

要创建自定义邮件解决方案，请实现 `Yiisoft\Mailer\MailerInterface` 和 `Yiisoft\Mailer\MessageInterface` 接口。

## 用于开发

对于本地或测试开发，您可以使用不发送电子邮件的邮件程序简化实现。该包提供了以下实现：

- `Yiisoft\Mailer\StubMailer` - 一个将消息存储在本地数组中的简单邮件程序。
- `Yiisoft\Mailer\FileMailer` - 一个将电子邮件消息保存为文件而不是发送它们的模拟邮件程序。
- `Yiisoft\Mailer\NullMailer` - 一个丢弃消息而不发送或存储它们的邮件程序。

要使用其中一种邮件程序，请在开发环境文件中配置它，例如：`environments/local/di.php`

```php
return [
    Yiisoft\Mailer\MailerInterface::class => Yiisoft\Mailer\StubMailer::class, // 或任何其他
];

```
