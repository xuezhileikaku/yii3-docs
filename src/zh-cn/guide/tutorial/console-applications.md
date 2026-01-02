# 控制台应用程序

控制台应用程序主要用于创建实用工具、后台处理和维护任务。

要在您的项目中获得控制台应用程序支持，请通过 composer 安装 `yiisoft/yii-console`：


```
composer require yiisoft/yii-console
```

安装完成后，您可以通过以下方式访问入口点

```
./yii
```

开箱即用，只有 `serve` 命令可用。它启动 PHP 内置 Web 服务器以在本地提供应用程序服务。

命令通过 `symfony/console` 执行。要创建您自己的控制台命令，您需要定义一个命令：

```php
<?php
namespace App\Command\Demo;

use Symfony\Component\Console\Attribute\AsCommand;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Style\SymfonyStyle;
use Yiisoft\Yii\Console\ExitCode;

#[AsCommand(
    name: 'demo:hello',
    description: 'Echoes hello',

)]
class HelloCommand extends Command
{
    public function configure(): void
    {
        $this
            ->setHelp('This command serves for demo purpose')
            ->addArgument('name', InputArgument::OPTIONAL, 'Name to greet', 'anonymous');
    }

    protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $io = new SymfonyStyle($input, $output);

        $name = $input->getArgument('name');
        $io->success("Hello, $name!");
        return ExitCode::OK;
    }
}
```

现在在 `config/params.php` 中注册命令：

```php
return [
    'console' => [
        'commands' => [
            'demo/hello' => App\Demo\HelloCommand::class,
        ],
    ],
];
```

完成后，可以通过以下方式执行命令

```
./yii demo:hello Alice
```


## 参考资料

- [Symfony Console 组件指南](https://symfony.com/doc/current/components/console.html)
