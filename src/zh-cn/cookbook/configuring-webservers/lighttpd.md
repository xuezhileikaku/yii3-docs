# 配置 Web 服务器：lighttpd

要使用 [lighttpd](https://www.lighttpd.net/) >= 1.4.24，请将 `index.php` 放在 web 根目录中并添加以下内容到配置：

```
url.rewrite-if-not-file = ("(.*)" => "/index.php/$0")
```
