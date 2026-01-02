# 014 — 文档

文档是 Yii 最重要的部分之一。

## 包文档

包的文档可以在 `README.md` 或 `docs/{language}/{type}` 中,其中 `{language}` 是语言代码,`{type}` 可以是 "guide"、"cookbook" 或其他。
通常,如果包用法或配置不平凡或需要翻译,则会存在 `docs`。

一些表明是时候创建 `docs` 的指标:

1. 需要翻译。
2. 存在许多主题。每个主题本身就很大。

如果 readme 的总长度小于大约 200 行,那么在 readme 中保留文档是可以的。

## 权威指南

权威指南,[yiisoft/docs/guide](https://github.com/yiisoft/docs/tree/master/guide/en)
旨在涵盖将包作为整体框架的使用。与包文档不同,它不专注于单个包,而是涵盖某些用例。

指南应该遵循 [Micosoft 风格指南](https://learn.microsoft.com/en-us/style-guide/welcome/)。

### 翻译

权威指南在 GitHub Action 中使用 [po4a](https://github.com/mquinson/po4a) 进行翻译。

翻译算法:

- 安装一个用于处理 `.po` 翻译文件的应用程序。例如,[Poedit](https://poedit.net/)、[Lokalize](https://apps.kde.org/ru/lokalize/)、
  [Gtranslator](https://wiki.gnome.org/Apps/Gtranslator) 或其他。
- 在 `_translations/guide/{lang}` 中查找您想要翻译的文件。请注意,如果源文件在子文件夹中,
  则子文件夹名称将附加到文件名并用下划线分隔,例如,
  要翻译 `guide/en/concept/aliases.md` 文件,请查找 `_translations/guide/{lang}/concept_aliases.md.po` 文件。
- 从所需本地化文件夹中用 `Poedit` 打开扩展名为 `.po` 的文件,
  例如 `_translations/guide/ru/intro_what-is-yii.md.po`。如果还没有本地化,请创建一个问题。
- 翻译必要的字符串并推送更改
- 打开拉取请求到主存储库

> [!CAUTION]
> 不要手动更改 `/guide/{lang}` 中文件中的翻译。

如果您已更改英语并想要更新翻译:

- 打开拉取请求到主存储库
- 在 GitHub Action 中的工作流程 `Update docs translation` 成功完成后拉取更新的分支
- 通过 `Poedit` 更新 `.po` 文件中的翻译
- 推送更改

## 块

块采用 [GitHub 警报格式](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#alerts):

```
> [!NOTE]
> 用户应该知道的有用信息,即使是在浏览内容时。

> [!TIP]
> 更好或更轻松地做事的有用建议。

> [!IMPORTANT]
> 用户实现目标需要知道的关键信息。

> [!WARNING]
> 需要用户立即注意以避免问题的紧急信息。

> [!CAUTION]
> 关于某些操作的风险或负面结果的警告。
```

翻译文档时,这些块指示符不应翻译。
保持它们完整不变,只翻译块内容。
要翻译块的标签,每个指南翻译应该有一个 `blocktypes.json` 文件,其中包含翻译。以下显示德语的示例:

```json
{
    "Note": "Hinweis",
    "Tip": "Tipp",
    "Important": "Wichtig",
    "Warning": "Achtung",
    "Caution": "Vorsicht"
}
```

## PHPDoc

如果 PHPDoc 没有添加任何描述它的内容,则不应添加它。以下是一个糟糕的示例:

```php
use Psr\Log\LoggerInterface;

/**
 * MyService 类
 */
final class MyService extends MyServiceBase
{
    /**
     * @var LoggerInterface 日志记录器
     */
    private LoggerInterface $logger;

    /**
     * MyService 构造函数。
     * @param LoggerInterface $logger
     */
    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    /**
     * @inheritDoc
     */
    public function doit(): bool
    {
        return parent::doit();
    }
}
```

如果存在 PHPDoc,应该描述为其添加的元素的目的。

## Readme 检查清单

每个包 readme 应该放在 `README.md` 中并包含以下内容:

- [ ] Logo。
- [ ] 包的简短描述。它有什么作用?
- [ ] 质量徽章(构建、代码覆盖率)。
- [ ] 截图(如果适用)。
- [ ] 要求。
- [ ] 安装。通常是 `composer require`。
- [ ] 入门。演示了一两个常见的用法示例。
- [ ] 配置。
- [ ] 贡献。它应该包含指向指南的链接。
- [ ] 运行测试。
- [ ] 许可证。
