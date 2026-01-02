# 005 — Yii 开发工具

对于 Yii3,包的数量显着增加,以实现更高的可重用性和独立发布。
为了简化框架本身的开发,我们创建了一个可从 [yiisoft/yii-dev-tool](https://github.com/yiisoft/yii-dev-tool) 获得的特殊工具。

```
$ ./yii-dev
  _   _  _  _
 | | | |(_)(_)  Development Tool
 | |_| || || |
  \__, ||_||_|  for Yii 3.0
  |___/

此工具有助于为 Yii 3 包设置开发环境。

用法:
  command [options] [arguments]

选项:
  -h, --help  显示此帮助消息

可用命令:
  checkout-branch  创建(如果不存在)并检出 git 分支
  commit           将更改添加并提交到每个包存储库
  install          安装包
  lint             根据 PSR12 标准检查包
  pull             从包存储库拉取更改
  push             将更改推送到包存储库
  replicate        将 replicate.php 中指定的文件复制到每个包
  status           显示包的 git 状态
  update           更新包
```

有许多可用命令。最重要的是 `install` 和 `update`。它们的作用是:

1. 安装/更新 [`packages.php`](https://github.com/yiisoft/yii-dev-tool/blob/master/packages.php) 中列出的所有包,
   如果指定了该列表中的单个包。
2. 对于每个安装的包,检查 `vendor` 目录中是否存在 `packages.php` 中列出的包。
   如果有任何,将包目录替换为指向另一个包源的符号链接。

因此,您将有许多相互使用的包,因此在开发期间无需 `git push` 和 `composer install` / `composer update`。

在其 README 中提供了[使用工具的详细示例](https://github.com/yiisoft/yii-dev-tool#usage-example)。
