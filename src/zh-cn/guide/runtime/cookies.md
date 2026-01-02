# Cookies

Cookie 用于在请求之间持久化数据，通过使用 HTTP 标头发送到客户端浏览器。
客户端在请求标头中将数据发送回服务器。因此，Cookie 适合存储少量数据，例如令牌或标志。

## 读取 Cookie

您可以从服务器请求中获取 Cookie 值，该请求可用作路由处理器（如控制器操作）的参数：

```php
private function actionProfile(\Psr\Http\Message\ServerRequestInterface $request)
{
    $cookieValues = $request->getCookieParams();
    $cookieValue = $cookieValues['cookieName'] ?? null;
    // ...
}
```

除了直接从服务器请求获取 Cookie 值外，您还可以使用 [yiisoft/request-provider](https://github.com/yiisoft/request-provider)
包，它通过 `\Yiisoft\RequestProvider\RequestCookieProvider` 提供了更结构化的 Cookie 处理方式。
这种方法可以简化您的代码并提高可读性。

以下是使用 `\Yiisoft\RequestProvider\RequestCookieProvider` 处理 Cookie 的示例：

```php
final readonly class MyService
{
    public function __construct(
        private \Yiisoft\RequestProvider\RequestCookieProvider $cookies
    ) {}

    public function go(): void
    {
        // 检查特定 Cookie 是否存在
        if ($this->cookies->has('foo')) {
            // 获取 Cookie 的值
            $fooValue = $this->cookies->get('foo');
            // 使用 Cookie 值执行某些操作
        }

        // 获取另一个 Cookie 值
        $barValue = $this->cookies->get('bar');
        // 使用 bar Cookie 值执行某些操作
    }
}
```

## 发送 Cookie

由于发送 Cookie 实际上是发送标头，但由于构建标头并不简单，因此提供了
`\Yiisoft\Cookies\Cookie` 类来帮助处理：

```php
$cookie = (new \Yiisoft\Cookies\Cookie('cookieName', 'value'))
    ->withPath('/')
    ->withDomain('yiiframework.com')
    ->withHttpOnly(true)
    ->withSecure(true)
    ->withSameSite(\Yiisoft\Cookies\Cookie::SAME_SITE_STRICT)
    ->withMaxAge(new \DateInterval('P7D'));

return $cookie->addToResponse($response);
```

构建 Cookie 后，调用 `addToResponse()` 并传入 `\Psr\Http\Message\ResponseInterface` 的实例，
以向其添加相应的 HTTP 标头。

## 签名和加密 Cookie

为了防止 Cookie 值被替换，该包提供了两种实现：

`Yiisoft\Cookies\CookieSigner` - 使用基于 Cookie 值和密钥的唯一前缀哈希对每个 Cookie 进行签名。
`Yiisoft\Cookies\CookieEncryptor` - 使用密钥对每个 Cookie 进行加密。

加密比签名更安全，但性能较低。

```php
$cookie = new \Yiisoft\Cookies\Cookie('identity', 'identityValue');

// 用于签名和验证 Cookie 的密钥。
$key = '0my1xVkjCJnD_q1yr6lUxcAdpDlTMwiU';

$signer = new \Yiisoft\Cookies\CookieSigner($key);
$encryptor = new \Yiisoft\Cookies\CookieEncryptor($key);

$signedCookie = $signer->sign($cookie);
$encryptedCookie = $encryptor->encrypt($cookie);
```

要验证并获取纯值，请使用 `validate()` 和 `decrypt()` 方法。

```php
$cookie = $signer->validate($signedCookie);
$cookie = $encryptor->decrypt($encryptedCookie);
```

如果 Cookie 值被篡改或之前未进行签名/加密，将抛出 `\RuntimeException`。
因此，如果您不确定 Cookie 值之前是否已签名/加密，
请首先分别使用 `isSigned()` 和 `isEncrypted()` 方法。

```php
if ($signer->isSigned($cookie)) {
    $cookie = $signer->validate($cookie);
}

if ($encryptor->isEncrypted($cookie)) {
    $cookie = $encryptor->decrypt($cookie);
}
```

如果您存储不应被用户更改的重要数据，则对 Cookie 值进行签名或加密是有意义的。

### 自动化加密和签名

要自动化 Cookie 值的加密/签名和解密/验证，请使用
`Yiisoft\Cookies\CookieMiddleware` 的实例，它是 [PSR-15](https://www.php-fig.org/psr/psr-15/) 中间件。

此中间件提供以下功能：

- 验证和解密请求中的 Cookie 参数值。
- 如果 Cookie 参数被篡改，则从请求中排除该参数并记录相关信息。
- 加密/签名 Cookie 值并替换响应中 `Set-Cookie` 标头中的原始值。

为了使中间件知道哪些 Cookie 的哪些值需要加密/签名，
必须将设置数组传递给其构造函数。数组键是 Cookie 名称模式，
值是 `CookieMiddleware::ENCRYPT` 或 `CookieMiddleware::SIGN` 的常量值。

```php
use Yiisoft\Cookies\CookieMiddleware;

$cookiesSettings = [
    // 与名称 `identity` 精确匹配。
    'identity' => CookieMiddleware::ENCRYPT,
    // 匹配下划线后的 1 到 9 的任何数字。
    'name_[1-9]' => CookieMiddleware::SIGN,
    // 匹配前缀后的任何字符串，包括
    // 空字符串，除了分隔符 "/" 和 "\"。
    'prefix*' => CookieMiddleware::SIGN,
];
```

有关使用通配符模式的更多信息，请参阅
[yiisoft/strings](https://github.com/yiisoft/strings#wildcardpattern-usage) 包。

创建和使用中间件：

```php
/**
 * @var \Psr\Http\Message\ServerRequestInterface $request
 * @var \Psr\Http\Server\RequestHandlerInterface $handler
 * @var \Psr\Log\LoggerInterface $logger
 */

// 用于签名和验证 Cookie 的密钥。
$key = '0my1xVkjCJnD_q1yr6lUxcAdpDlTMwiU';
$signer = new \Yiisoft\Cookies\CookieSigner($key);
$encryptor = new \Yiisoft\Cookies\CookieEncryptor($key);

$cookiesSettings = [
    'identity' => \Yiisoft\Cookies\CookieMiddleware::ENCRYPT,
    'session' => \Yiisoft\Cookies\CookieMiddleware::SIGN,
];

$middleware = new \Yiisoft\Cookies\CookieMiddleware(
    $logger
    $encryptor,
    $signer,
    $cookiesSettings,
);

// 请求中的 Cookie 参数值被解密/验证。
// Cookie 值被加密/签名并附加到响应。
$response = $middleware->process($request, $handler);
```

如果 `$cookiesSettings` 数组为空，则不会对任何 Cookie 进行加密和签名。

## Cookie 安全性

您应该将每个 Cookie 配置为安全的。重要的安全设置包括：

- `httpOnly`。将其设置为 `true` 可以防止 JavaScript 访问 Cookie 值。
- `secure`。将其设置为 `true` 可以防止通过 `HTTP` 发送 Cookie。它将仅通过 `HTTPS` 发送。
- `sameSite`，如果设置为 `SAME_SITE_LAX` 或 `SAME_SITE_STRICT`，将防止在跨站点
  浏览上下文中发送 Cookie。`SAME_SITE_LAX` 将在易受 CSRF 攻击的请求方法（例如 POST、PUT、
  PATCH 等）期间阻止 Cookie 发送。`SAME_SITE_STRICT` 将阻止所有方法的 Cookie 发送。
- 对 Cookie 值进行签名或加密，以在值中的数据不应被篡改时防止值被伪造。
