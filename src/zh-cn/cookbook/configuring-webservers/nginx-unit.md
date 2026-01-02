# 配置 Web 服务器：NGINX Unit

使用 [NGINX Unit](https://unit.nginx.org/) 和 PHP 语言模块运行基于 Yii 的应用程序。
这是一个示例配置。

```json
{
    "listeners": {
        "*:80": {
            "pass": "routes/yii"
        }
    },

    "routes": {
        "yii": [
            {
                "match": {
                    "uri": [
                        "!/assets/*",
                        "*.php",
                        "*.php/*"
                    ]
                },

                "action": {
                    "pass": "applications/yii/direct"
                }
            },
            {
                "action": {
                    "share": "/path/to/app/public/",
                    "fallback": {
                        "pass": "applications/yii/index"
                    }
                }
            }
        ]
    },

    "applications": {
        "yii": {
            "type": "php",
            "user": "www-data",
            "environment": {
                "APP_ENV": "dev"
            },
            "targets": {
                "direct": {
                    "root": "/path/to/app/public/"
                },

                "index": {
                    "root": "/path/to/app/public/",
                    "script": "index.php"
                }
            }
        }
    }
}
```

您也可以[设置](https://unit.nginx.org/configuration/#php)您的 PHP 环境或在同一配置中提供自定义 `php.ini`。

在上面，请注意 `environment` 的用法。由于 Yii3 应用程序模板使用环境变量，这是设置它们的地方之一。在生产环境中，请记住将 `APP_ENV` 设置为 `prod`。
