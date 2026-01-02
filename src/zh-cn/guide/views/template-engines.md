# 模板引擎

Yii3 通过灵活的渲染器系统支持多种模板引擎。默认情况下，PHP 用作模板引擎，但您可以轻松添加对其他引擎（如 Twig）的支持或创建自己的自定义渲染器。

PHP 模板在 "[视图](view.md)" 指南部分中有介绍。

## Twig 模板引擎

Twig 是一个现代模板引擎，提供了更友好的设计器语法。要在 Yii3 应用程序中使用 Twig，您需要安装 Twig 扩展。

```bash
composer require yiisoft/view-twig
```

现在您可以使用 `.twig` 模板。例如，`views/site/about.twig`：

```twig
{# 变量类型提示以支持 IDE #}
{# @var user \App\Entity\User #}
{# @var posts \App\Entity\Post[] #}

<div class="user-profile">
    <h1>{{ user.name }}</h1>
    <p>Email: {{ user.email }}</p>

    {% if posts is not empty %}
        <h2>Recent Posts</h2>
        <ul class="posts">
            {% for post in posts %}
                <li>
                    <h3>{{ post.title }}</h3>
                    <p>{{ post.excerpt }}</p>
                    <time>{{ post.publishedAt|date('F j, Y') }}</time>
                </li>
            {% endfor %}
        </ul>
    {% else %}
        <p>No posts available.</p>
    {% endif %}
</div>
```

### Twig 功能

**自动转义**：Twig 自动为 HTML 上下文转义变量：

```twig
{# 自动转义 #}
<h1>{{ title }}</h1>

{# 原始输出（小心使用）#}
<div>{{ content|raw }}</div>
```

**过滤器和函数**：Twig 提供了许多内置的过滤器和函数：

```twig
{# 日期格式化 #}
<time>{{ post.createdAt|date('Y-m-d H:i') }}</time>

{# 字符串操作 #}
<p>{{ description|truncate(100) }}</p>

{# URL 生成 #}
<a href="{{ path('user.profile', {'id': user.id}) }}">Profile</a>
```

**模板继承**：Twig 支持模板继承：

**views/layout/main.twig**
```twig
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Default Title{% endblock %}</title>
</head>
<body>
    <main>
        {% block content %}{% endblock %}
    </main>
</body>
</html>
```

**views/site/about.twig**
```twig
{% extends "layout/main.twig" %}

{% block title %}About Us{% endblock %}

{% block content %}
    <h1>About Our Company</h1>
    <p>Welcome to our website!</p>
{% endblock %}
```

### 渲染 Twig 模板

使用 Twig 模板与使用 PHP 模板的方式相同：

```php
// 在您的控制器中
public function about(): ResponseInterface
{
    return $this->viewRenderer->render('about.twig', [
        'user' => $this->getCurrentUser(),
        'posts' => $this->getRecentPosts(),
    ]);
}
```

## 自定义模板引擎

您可以通过实现 `TemplateRendererInterface` 来创建自定义模板引擎：

```php
<?php

declare(strict_types=1);

namespace App\View;

use Yiisoft\View\TemplateRendererInterface;

final class MarkdownRenderer implements TemplateRendererInterface
{
    public function __construct(
        private MarkdownParserInterface $parser,
    ) {}

    public function render(string $template, array $parameters = []): string
    {
        $content = file_get_contents($template);

        // 用参数替换占位符
        foreach ($parameters as $key => $value) {
            $content = str_replace("{{$key}}", (string) $value, $content);
        }

        return $this->parser->parse($content);
    }
}
```

注册您的自定义渲染器：

```php
use Yiisoft\Container\Reference;

// 在配置中
'yiisoft/view' => [
    'renderers' => [
        'md' => Reference::to(App\View\MarkdownRenderer::class),
    ],
],
```

现在您可以使用 `.md` 模板文件：

**views/content/help.md**
```markdown
# Help: {{title}}

Welcome, {{username}}!

This is a Markdown template with **bold** and *italic* text.

- Feature 1
- Feature 2
- Feature 3
```

## 选择正确的模板引擎

**使用 PHP 模板时：**
- 您需要最大的灵活性和性能
- 您的团队熟悉 PHP
- 您想利用现有的 PHP 知识
- 您需要在模板中使用复杂的逻辑（尽管这应该最小化）

**使用 Twig 模板时：**
- 您希望在逻辑和表示之间严格分离
- 您与更喜欢简洁语法的设计师合作
- 您需要自动转义和安全功能
- 您想要模板继承和高级功能

**使用自定义模板时：**
- 您有 PHP 或 Twig 无法满足的特定要求
- 您正在处理专门的内容格式
- 您需要与外部模板系统集成

## 最佳实践

1. **保持模板简单**：将复杂的逻辑移到控制器或服务中
2. **始终转义输出**：通过正确转义变量来防止 XSS 攻击
3. **使用有意义的名称**：清晰命名模板和变量
4. **组织模板**：将相关模板分组在子目录中
5. **记录变量**：始终添加类型提示以获得更好的 IDE 支持
6. **避免业务逻辑**：将业务逻辑保留在模型和服务中
