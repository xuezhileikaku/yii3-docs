# 配置 Web 服务器：Apache

在 Apache 的 `httpd.conf` 文件或虚拟主机配置中使用以下配置。请注意，您应该将 `path/to/app/public` 替换为 `app/public` 的实际路径。

```apache
# 将文档根目录设置为 "app/public"
DocumentRoot "path/to/app/public"

<Directory "path/to/app/public">
    # 使用 mod_rewrite 支持 URL 美化
    RewriteEngine on

    # 如果 UrlManager 中的 $showScriptName 为 false，则不允许通过脚本名称访问 URL
    RewriteRule ^index.php/ - [L,R=404]

    # 如果目录或文件存在，则直接使用请求
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d

    # 否则将请求转发到 index.php
    RewriteRule . index.php

    SetEnv APP_ENV dev

    # ...其他设置...
</Directory>
```

如果您有 `AllowOverride All`，您可以添加 `.htaccess` 文件并使用以下配置，而不是使用 `httpd.conf`：

```apache
# 使用 mod_rewrite 支持 URL 美化
RewriteEngine on

# 如果 UrlManager 中的 $showScriptName 为 false，则不允许通过脚本名称访问 URL
RewriteRule ^index.php/ - [L,R=404]

# 如果目录或文件存在，则直接使用请求
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d

# 否则将请求转发到 index.php
RewriteRule . index.php

SetEnv APP_ENV dev

# ...其他设置...
```

在上面，请注意 `SetEnv` 的用法。由于 Yii3 应用程序模板使用环境变量，这是设置它们的地方之一。在生产环境中，请记住将 `APP_ENV` 设置为 `prod`。
