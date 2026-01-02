# 请求

HTTP 请求具有方法、URI、一组标头和主体：

```
POST /contact HTTP/1.1
Host: example.org
Accept-Language: en-us
Accept-Encoding: gzip, deflate

{
    "subject": "Hello",
    "body": "Hello there, we need to build Yii application together!"
}
```

方法是 `POST`，URI 是 `/contact`。
额外的标头指定了主机、首选语言和编码。
主体可以是任何东西。
在这种情况下，它是一个 JSON 负载。

Yii 使用 [PSR-7 `ServerRequest`](https://www.php-fig.org/psr/psr-7/) 作为请求表示。
该对象在控制器操作和其他类型的中间件中可用：

```php
public function view(ServerRequestInterface $request): ResponseInterface
{
    // ...
}
```

## 方法

可以从请求对象获取方法：

```php
$method = $request->getMethod();
```

通常是以下之一：

- GET
- POST
- PUT
- DELETE
- HEAD
- PATCH
- OPTIONS

如果您想确保请求方法是某种类型，可以使用一个带有方法名称的特殊类：

```php
use Yiisoft\Http\Method;

if ($request->getMethod() === Method::POST) {
    // 方法是 POST
}
```

## URI

URI 具有以下内容：

- 方案（`http`、`https`）
- 主机（`yiiframework.com`）
- 端口（`80`、`443`）
- 路径（`/posts/1`）
- 查询字符串（`page=1&sort=+id`）
- 片段（`#anchor`）

您可以像以下那样从请求中获取 `UriInterface`：

```php
$uri = $request->getUri();
```

然后您可以从其方法获取各种详细信息：

- `getScheme()`
- `getAuthority()`
- `getUserInfo()`
- `getHost()`
- `getPort()`
- `getPath()`
- `getQuery()`
- `getFragment()`

## 标头

有各种方法可以检查请求标头。要将所有标头作为数组获取：

```php
$headers = $request->getHeaders();
foreach ($headers as $name => $values) {
   // ...
}
```

要获取单个标头：

```php
$values = $request->getHeader('Accept-Encoding');
```


此外，您可以将值作为逗号分隔的字符串而不是数组获取。
如果标头具有单个值，这特别方便：

```php
if ($request->getHeaderLine('X-Requested-With') === 'XMLHttpRequest') {
    // 这是使用 jQuery 发出的 AJAX 请求。
    // 请注意，标头的存在和名称可能因使用的库而异。
}
```

要检查请求中是否存在标头：

```php
if ($request->hasHeader('Accept-Encoding')) {
    // ...
}
```

## 主体

有两种方法可以获取主体内容。第一种是原样获取主体而不进行解析：

```php
$body = $request->getBody();
```

`$body` 将是 `Psr\Http\Message\StreamInterface` 的实例。

此外，您可以获取解析后的主体：

```php
$bodyParameters = $request->getParsedBody();
```

解析取决于 PSR-7 实现，并且可能需要中间件来处理自定义主体格式。

```php
<?php
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;

final readonly class JsonBodyParserMiddleware implements MiddlewareInterface
{
    public function process(Request $request, RequestHandler $next): Response
    {
        $contentType = $request->getHeaderLine('Content-Type');

        if (strpos($contentType, 'application/json') !== false) {
            $body = $request->getBody();
            $parsedBody = $this->parse($body);
            $request = $request->withParsedBody($parsedBody);

        }

        return $next->handle($request);
    }
}
```

## 文件上传

用户从具有 `enctype` 属性等于 `multipart/form-data` 的表单提交的上传文件
通过特殊的请求方法处理：

```php
$files = $request->getUploadedFiles();
foreach ($files as $file) {
    if ($file->getError() === UPLOAD_ERR_OK) {
        $file->moveTo('path/to/uploads/new_filename.ext');
    }
}
```

## 属性

应用程序中间件可以使用 `withAttribute()` 方法设置自定义请求属性。
您可以使用 `getAttribute()` 获取这些属性。
