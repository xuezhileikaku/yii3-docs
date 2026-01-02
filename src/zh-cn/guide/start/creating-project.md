# 创建项目

在本指南中，我们将提供 [Docker](https://docs.docker.com/get-started/get-docker/) 和本地安装所有内容后使用内置开发服务器的命令。

> [!NOTE]
> 如果你想使用其他 Web 服务器，
> 请参阅["配置 Web 服务器"](../../cookbook/configuring-webservers/general.md)。

我们建议从一个项目模板开始，它是一个实现了某些基本功能的最小可运行 Yii 项目。它可以作为你项目的良好起点。

你可以使用 [Composer](https://getcomposer.org) 包管理器从模板创建一个新项目：

```sh
composer create-project yiisoft/app your_project
```

Docker 用户可以运行以下命令：

```sh
docker run --rm -it -v "$(pwd):/app" composer/composer create-project yiisoft/app your_project
sudo chown -R $(id -u):$(id -g) your_project
make composer update
```

这会将最新稳定版本的 Yii 项目模板安装到一个名为 `your_project` 的目录中。如果你愿意，可以选择不同的目录名。

> [!TIP]
> 如果你想安装 Yii 的最新开发版本，可以在命令中添加 `--stability=dev`。
> 请不要在生产环境中使用 Yii 的开发版本，因为它可能会破坏你运行的代码。

进入新创建的目录并运行：

```sh
APP_ENV=dev ./yii serve --port=80
```

对于 Docker 用户，运行：

```sh
make up
```

在浏览器中打开 URL `http://localhost/`。

> [!NOTE]
> HTTP 服务器监听端口 80。如果该端口已被占用，可以通过 `--port` 指定端口，或者在 Docker 的情况下，
> 在 `docker/.env` 文件中指定 `DEV_PORT`。

![Yii 安装成功](/images/guide/start/app-installed.png)
